---
layout: post
title: "Is Ruby Dying? Evaluating a Programming Language's Popularity using Probabilistic Programming"
date: 2017-06-20 18:00:00 +0200
---

Last week I had a conversation about the question "Is the Ruby programming language losing popularity". It was a fairly open-ended question and so understandably we didn't come to a conclusion. However, shortly afterwards I came up with a similar question that I could answer using my latest learnings about probabilistic programming.


## Probabilistic Programming?

I have to admit that the term *probabilistic programming* requires an explanation. To quote *Cameron Davidson-Pilon*, the author of the [excellent book "Bayesian Methods for Hackers"](https://github.com/CamDavidsonPilon/Probabilistic-Programming-and-Bayesian-Methods-for-Hackers), *probabilistic programming is &hellip;*

> "an unfortunate misnomer that invokes ideas of randomly-generated code and has likely confused and frightened users away from this field. The code is not random; it is probabilistic in the sense that we create probability models using programming variables as the model's components."

In other words: We're simply writing programs about probability models. The computer then executes these programs and finds answers to some of our questions.

We'll learn more about this soon, so let's come back to our question.


## (Re)phrasing the Question

Judging if the *Ruby* programming language is losing popularity is a fairly complex question that would be hard to answer. Instead, we can substitute it:

*Does my local Ruby meetup currently have significantly less attendees than in the past?*

Admittedly, an answer to this question doesn't really answer the original one, but it might become a good argument for or against the original it. Being able to confidently say *"my local Ruby meetup has not lost a significant amount of attendees"* might be convincing, especially if you can make this statement about all *Ruby* meetups.


## How We'll Answer the Question


We'll do most of the work with the extensive [PyMC](https://pymc-devs.github.io/pymc/) library. According to the [offical website](https://pymc-devs.github.io/pymc/README.html):

> "PyMC is a python module that implements Bayesian statistical models and fitting algorithms, including Markov chain Monte Carlo."


The first thing that catches the eye is "*Bayesian* statistical models". Bayes was a famous mathematician whose name you might have first heard in high school in combination with [Bayes' theorem](https://en.wikipedia.org/wiki/Bayes%27_theorem). In this post we're particularly interested in [Bayesian statistics](https://en.wikipedia.org/wiki/Bayesian_statistics).

Bayesian statistics is a branch of statistics that allows us to first look at data and then explore how this data might have been created in the first place. 

Now, instead of doing all the math by hand, we'll let PyMC do the work for us. We'll simply provide it with our dataset and a couple of rules. PyMC will then use a *fitting algorithm* called [Markov chain Monte Carlo simulation](https://en.wikipedia.org/wiki/Markov_chain_Monte_Carlo). As a result we'll get some values which - in combination with the provided rules - can be used to generate our dataset. It is then up to us to *interpret* those in regard to our question.

Keeping this rough overview in mind, let's jump right in and get our hands dirty!


## Setting Up Your Machine

We'll be using [Python 3.6.1](https://www.python.org) and [PyMC](https://pymc-devs.github.io/pymc/) together with the libraries [numpy](http://numpy.org), [matplotlib](http://matplotlib.org) and [scipy](http://scipy.org). If you already have `Python`, you can install those using the following command:

    pip install pymc numpy matplotlib scipy

Next, you'll need a dataset to work with. For now I'd recommend to [download the same dataset as I'm using in this example](/downloads/meetup_attendees.csv). Later on, you can try this with your own meetup datasets. 

Finally, choose the text editor of your choice (*any text editor will do, but if you like interactive programming you should check out [jupyter notebooks](http://jupyter.org)!*).


## Having a First Look at our Data

Let's have a look at the first couple of lines of our dataset to understand its structure. We can use a few lines of Python for this:

```python
with open('./meetup_attendees.csv') as file:
    for line in list(file)[0:3]:
        print(line, end='')
```

```
> november-meetup-2012, 22
> december-meetup-2012, 35
> january-meetup-2013, 39
```

This dataset has a very simple structure; each line represents a single meetup and contains a meetup identifier and the number of attendees, separated by a comma. The list of meetups is ordered by date and begins with the oldest one.
Let's encode that knowledge into our program:


```python
def parse_line(line):
    event, attendees = line.split(',')
    parsed_event, parsed_attendees = event.strip(), int(attendees)
    return (parsed_event, parsed_attendees)

with open('./meetup_attendees.csv') as file:
    meetups = list(map(parse_line, file))

num_meetups = len(meetups)
first_meetup = meetups[0]
last_meetup = meetups[-1]

print('There have been {} meetups:'.format(num_meetups))
print('- the first one in "{}" with {} attendees'.format(first_meetup[0], first_meetup[1]))
print('- the last one in "{}" with {} attendees'.format(last_meetup[0], last_meetup[1]))
```

```
> There have been 56 meetups:
> - the first one in "november-meetup-2012" with 22 attendees
> - the last one in "june-meetup-2017" with 27 attendees
```

Great! One of the best ways to get a first understanding of a dataset is to visualise it. Here we'll plot a barchart using the [Matplotlib](http://matplotlib.org) library.

```python
import matplotlib.pyplot as plt

attendees = list(map(lambda meetup: meetup[1], meetups))

plt.bar(range(0, len(attendees)), attendees)
plt.title('Number of attendees over time')
plt.xlabel('Time')
plt.ylabel('Attendees')
plt.show()
```

![Number of attendees over time](/images/attendees/plot_01_attendees.png)

What do we see in this chart? Here are a couple of observations:

- The amount of attendees was varying from meetup to meetup.
- We can cleary see a rising number of attendees at the beginning of the meetup. 
- Then, it looks as if the number of attendees stayed consistent for a while.
- During the last meetups, we can see the number of attendees declining again.

This is interesting! I'd even say that this chart is able to give us a good first understanding of the dataset. From looking again at our observations I'd say that the number of attendees has indeed been declining. But this is nothing but a guess. To make a solid argument out of it, we'll need some form of proof.

The best way to get such a proof is by formulating a model and then letting PyMC validate it!


## Coming up with a Model

Let's suppose that the popularity of the meetup changed twice. We can formulate this in a different way: Assume that there were two points in time tau_1, tau_2 in which the popularity of the meetup changed.

In that case there must be three groups of meetups:

- all meetups that happened before tau_1, 
- meetups that happened after tau_1, but before tau_2
- the remaining meetups that happened after tau_2

Furthermore, the three groups must differ from each other. If they weren't, the popularity of the meetup wouldn't have changed and our assumption would be wrong.

In our case, a good way to distinguish the groups from each other would be the mean value of attendees.

## The Mean Value of Attendees

We'll get the mean value of attendees for each group by summing the number of attendees for all meetups and deviding it by the number of meetups.

Just as an exercise, this is the mean value of attendees for all meetups:


```python
mean_attendees = sum(attendees) / len(attendees)
print('The mean value of attendees is {}.'.format(mean_attendees))
```

```
> The mean value of attendees is 55.19642857142857.
```

So, on average each meetup had about 55 participants. Here's a visualisation of the mean value of attendees on top of the dataset:

```python
list_of_mean_attendees = [mean_attendees for _ in attendees]
plt.bar(range(0, len(attendees)), attendees)
plt.plot(list_of_mean_attendees, label="Mean", color='#fe1212')
plt.title('Number of attendees over time with mean')
plt.xlabel('Time')
plt.ylabel('Attendees')
plt.legend()
plt.show()
```

![Number of attendees over time with mean](/images/attendees/plot_02_attendees_mean.png)


## Back to our Model

Ok, now as we've seen how the mean value looks like, we can get back to formulating our model.

As I mentioned before, we'll assume that the popularity of our meetup changed on two points in time, tau_1, tau_2. This separates the meetups into three groups; each group having its own mean value of attendees mean_1, mean_2, mean_3.

What are those mean values good for? We can compare them to understand how they changed over time:

- If the mean values increased, we can say that the meetup gained popularity.
- If they decreased, we can say that the meetup lost popularity.

Great, we've done it! This reformulation of our original question is finally simple enough for the computer to answer.

## Making a First Guess

To roughly understand what the computer will be doing, let's make a first, uneducated guess and look at the results:

First, we'll naively guess that the behaviour changed after the 25th and after the 30th meetup.

```python
from statistics import mean

tau_1 = 25
tau_2 = 30

group_1 = attendees[0:tau_1]
group_2 = attendees[tau_1:tau_2]
group_3 = attendees[tau_2:]

print('Group 1 has a mean value of {}'.format(mean(group_1)))
print('Group 2 has a mean value of {}'.format(mean(group_2)))
print('Group 3 has a mean value of {}'.format(mean(group_3)))
```

```
> Group 1 has a mean value of 54.6
> Group 2 has a mean value of 70.2
> Group 3 has a mean value of 52.88461538461539
```

Here's how it looks like in a visualisation:

```python
mean_attendees_1 = [mean(group_1) for _ in group_1]
mean_attendees_2 = [mean(group_2) for _ in group_2]
mean_attendees_3 = [mean(group_3) for _ in group_3]

mean_attendees_groups = mean_attendees_1 + mean_attendees_2 + mean_attendees_3

plt.bar(range(0, len(attendees)), attendees)
plt.vlines(tau_1, 0, 100, linestyle='dashed', label=r'$\tau_1$')
plt.vlines(tau_2, 0, 100, linestyle='dashed', label=r'$\tau_2$')
plt.plot(mean_attendees_groups, label="Mean", color='#fe1212')
plt.title('Number of attendees over time with means')
plt.xlabel('Time')
plt.ylabel('Attendees')
plt.legend()
plt.show()
```

![Number of attendees over time with means](/images/attendees/plot_03_attendees_means.png)


We can observe three things:

- The mean values of the first and third group are quite similar to the overal mean value.
- The mean value doesn't really fit the data of the first and last group
- Only in center we have a higher mean value

All in all, I'd say that our naive guess wasn't that great. The mean values don't fit the actual dataset. What a shame.

## Finding Answers With PyMC

Luckily, we can use PyMC to perform thousands of such guesses in no time. Even better, PyMC uses a sophisticated algorithm to reduce the amount of guesses required. In the end however, it will give us nothing more than the two points in time at which the behaviour most likely changed.

Let's finally see the actual PyMC code:


```python
import pymc
import pymc.distributions as dis

tau_1 = dis.DiscreteUniform('tau_1', 0, len(meetups))
tau_2 = dis.DiscreteUniform('tau_2', 0, len(meetups))

# Some magic here, don't worry too much about it (for now)
mean_1 = dis.DiscreteUniform('mean_1', 0, max(attendees))
mean_2 = dis.DiscreteUniform('mean_2', 0, max(attendees))
mean_3 = dis.DiscreteUniform('mean_3', 0, max(attendees))

@pymc.deterministic
def means(tau_1=tau_1, tau_2=tau_2, mean_1=mean_1, mean_2=mean_2, mean_3=mean_3):
    means = [0 for _ in attendees]
    means[:tau_1] = [mean_1 for _ in means[:tau_1]]
    means[tau_1:tau_2] = [mean_2 for _ in means[tau_1:tau_2]]
    means[tau_2:] = [mean_3 for _ in means[tau_2:]]
    return means

observations = dis.Poisson('observations', means, value=attendees, observed=True)

model = pymc.MCMC([observations, means, mean_1, mean_2, mean_3, tau_1, tau_2])

model.sample(40000)

# Don't worry about the mean() calls either (for now)
tau_1_samples = model.trace('tau_1')[:].mean()
tau_2_samples = model.trace('tau_2')[:].mean()
mean_1_samples = model.trace('mean_1')[:].mean()
mean_2_samples = model.trace('mean_2')[:].mean()
mean_3_samples = model.trace('mean_3')[:].mean()

tau_1 = int(tau_1_samples.mean())
tau_2 = int(tau_2_samples.mean())
mean_1 = mean_1_samples.mean()
mean_2 = mean_2_samples.mean()
mean_3 = mean_3_samples.mean()
```

```
>  [-----------------100%-----------------] 40000 of 40000 complete in 11.4 sec
```

Don't worry too much about the code. It's really just our way of setting up PyMC. There are two topics that you might want to research on your own:

- Distributions really are the most important modelling tool in probabalistic programming. The [Poisson distribution](https://en.wikipedia.org/wiki/Poisson_distribution#Mean) is one of them.
- To estimate the values for our parameters, we (again) calculate the mean value of a huge list of samples. If you're interested in the exact distribution, try to plot some histograms.

Are you also excited to see the results? Here they are:

```python
print('The first split occured after meetup {}'.format(tau_1))
print('The second split occured after meetup {}'.format(tau_2))
print('In the first group we have {} attendees on average'.format(mean_1))
print('In the second group we have {} attendees on average'.format(mean_2))
print('In the third group we have {} attendees on average'.format(mean_3))
```

```
> The first split occured after meetup 9
> The second split occured after meetup 37
> In the first group we have 32.901975 attendees on average
> In the second group we have 59.98695 attendees on average
> In the third group we have 47.284025 attendees on average
```

And here's the visualisation you've been waiting for. I promise, it does fit the data surprisingly well.


```python
group_1 = attendees[0:tau_1]
group_2 = attendees[tau_1:tau_2]
group_3 = attendees[tau_2:]

mean_attendees_1 = [mean_1 for _ in group_1]
mean_attendees_2 = [mean_2 for _ in group_2]
mean_attendees_3 = [mean_3 for _ in group_3]

mean_attendees_groups = mean_attendees_1 + mean_attendees_2 + mean_attendees_3

plt.bar(range(0, len(attendees)), attendees)
plt.vlines(tau_1, 0, 100, linestyle='dashed', label=r'$\tau_1$')
plt.vlines(tau_2, 0, 100, linestyle='dashed', label=r'$\tau_2$')
plt.plot(mean_attendees_groups, label="Mean", color='#fe1212')
plt.title('Number of attendees over time with means')
plt.xlabel('Time')
plt.ylabel('Attendees')
plt.legend()
plt.show()
```

![Attendees over time with means (correct)](/images/attendees/plot_04_attendees_means.png)

## Interpreting the Results

Well, what does all this tell us? First of all, we see that the meetup gained in popularity at the beginning. Secondly, we see that it stayed quite popular for a while. And finally, the meetup lost a bit in popularity.

To summarize, I'd claim that **yes, my local ruby meetup clearly has significantly less attendees than in the past**. What does that tell us about Ruby? Is Ruby dying? We can't say. What a pitty.

## Srsly? What now?


Now you think: Wow! Why did I read through all this just to have learned nothing in the end? I have to admit, you didn't learn anything about Ruby's destiny today. But, with a bit of luck, you've learned quite a lot about probabilistic programming.

You might have a lot of questions by now, for example:

- What are distributions and what are they good for?
- How does PyMC actually produce these results for us?
- How good is our data?
- Why two points of change? Why not one? Or three?

Answering those questions would go far beyond the scope of this article.

Luckily, there's this great book [Bayesian Methods for Hackers by Cameron Davidson-Pilon](https://github.com/CamDavidsonPilon/Probabilistic-Programming-and-Bayesian-Methods-for-Hackers).



It will give you answers to all these questions and will even go far beyond what I presented here.