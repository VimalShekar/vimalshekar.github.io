---
layout: post
title: Machine Learning Basics Part 1
date: 2018-01-28
category: MachineLearning
---
# Machine Learning Basics Part 1

"*Technology is like the "Drink me" potion from Alice in Wonderland*"

At some point in my career - I was engaged in assembly level debugging, and started off this blog to save some lessons for my self and others. Now, I'm at another point in my career - (in another country too) and engaged in Cloud automation and Machine Learning.

The shift almost feels like moving from observing molecular structures under a microscope to observing distant planets looking for life. I almost feel like Alice in Wonderland - who took the Drink Me potion. But I cant complain. Technology has grown leaps and bounds in my lifetime and Its a humbling feeling to know that I could witness, learn and hopefully be a part of this.

So here I am - getting started on this blog series to share my learnings for my future self and hopefully others.

We start off with an FAQ:

## What is Machine Learning?

When you first hear it, you may think about artificial intelligence and humanoids. But both of those are fields of their own.

A Text book definition first:
A computer is said to learn from experience E, with respect to some task T and some performance measure P - if its performance P on task T improves with experience E. (Tom Mitchell - 1998)

ML is essentially the science and art of getting a system(aka machine) to learn and adapt to different inputs. We do not have to explicitely program the machine to handle each type of input, instead the machine has a learning element that adapts itself based on data is has seen and improvises on that to handle an input that it has not seen.

## Are there different types?

ML is classified in lots of ways. The most prominent classification is probably:

1. Supervised learning
2. Unsupervised learning

## What is supervised learning?

In Supervised learning, we teach the system to behave by providing training data(aka labelled data/samples). Once the machine has seen this, it learns to adapts itself so that it can predict the output on new inputs. 

Imagine training your dog to fetch a stick.

## What is unsupervised learning?

You guessed it, here the system is not given any training data with expected results. It just learns based on the given input dataset and learns to uncover hidden patterns and structures within the dataset.

Imagine you trying to solve a new puzzle the very first time.

Supervised and Unsupervised is just the tip of the iceberg. There are several more classifications, we'll pick on them as we go.
![Types of ML problems](https://cdn-images-1.medium.com/max/1600/1*ZCeOEBhvEVLmwCh7vr2RVA.png)

## What are Labels and Features?

A Label is the thing we are predicting. In other words, it is the output we expect to see for a given set of inputs.
A feature is the input given to the system, in other words the variables on which the output depends.

Consider a simple algebraic expression:
y = a1 x1 + a2 x2 + a3 x3 + .... + an xn
    y is the output or label
    x1, x2 ... xn are inputs or features of the system.
    a1, a2 ... 1n are constants

Consider another example of a spam prediction system. The inputs or features here would be the words in the email, sender's address, time of day, subject of the email etc. The output or label would be the decision taken by the system(Spam or not Spam).

## What is training data/samples?

In the case of supervised systems, the system examines labelled data to figure out the pattern and then gets ready for unlabelled data. When I say labelled data - I'm referring to both features (x1, x2 ... xn) and the output label (y) for that given instance of inputs.
    {features, label}: (x, y)

## What is a model or Hypothesis?

A model or hypothesis is the relationship between features and the label (input and the output). In classical Control theory - you could call this the transfer function of the system.

Once the system sees a number of labelled examples, it builds a model. This is called training the model.

Once the model is ready, the system uses the model to predict the label on an unlabelled sample.

That covers two stages of the model's life - Training and Inference.

## What is training and inference?

**Training** means creating or learning the model. That is, you show the model labeled examples and enable the model to gradually learn the relationships between features and label.

**Inference** means applying the trained model to unlabeled examples. That is, you use the trained model to make useful predictions (y'). 

## Regression and Classification - what's the difference?

The first steps in ML is to identify what type of problem you are trying to solve, and whether you have training data or not. If you do, then you treat this as a Supervised learning problem. Supervised learning problems are further classified into regression and classification problems.

A classification model predicts distinct or discrete values. For example, classification models make predictions that answer questions like the following:

- Is a given email message spam or not spam?
- Is this an image of a dog, a cat, or a hamster?

A regression model predicts continuous values. For example, regression models make predictions that answer questions like the following:

- What is the value of a house in Bangalore?
- What is the average value of temperature in a city for a given month?

## How do I go about learning ML?

Think of this hypothetical problem:
The price of a kg of mangoes is 5$, 2 kg is 10$ and 5 Kg is 25$. From this data, we could make an inference that the price of a kilo of mangoes is 5$, and the price follows this pattern:
Price in $ = 5 * (Number of Kgs purchased)

From that basic inference you're now able to predict the price of any given weight of mangoes. 

A ML model also learns in the same way. What we've just accomplished is a simple ML model called Linear regression. Congratulations, Look at what you've accomplished!

Well, that's it for now - But, there are many more where that came from. In the next blog, we'll look at linear regression in more detail.