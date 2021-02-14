---
layout: default
title: "What Can Machine Learning Do in the Search for Prime Numbers?"
permalink: /PrimeDetection-ML/
---

# What Can Machine Learning Do in the Search for Prime Numbers?

Machine learning has proven to be useful in a broad number of fields. Genuinely difficult problems such as image recognition and handwriting recognition are quickly approaching that status of a “Solved Problem”. However, the techniques that are used to approach this problem may be useful in other fields, such as number theory. It is well established that it is difficult to determine if a number is truly prime or not. That is, does a given positive integer, n, have only two distinct factors, 1 and n? If so, then the number is a prime number. While there are many algorithms and tricks that can rule out large amounts of numbers as being composite (aka, not prime), this still does not fully create an algorithm that answers the question of is a number prime or not. Many numbers which are in fact composite will pass through the filter.

## The Dataset

This is one of the cases where it is better to make a dataset from scratch than to search for one on the internet. There are lists of prime numbers out there, (with the largest one I was able to find, listing the first 50 million prime numbers), but there are some limitations with just downloading and using a pre-built dataset.

The first problem is that the pre-built datasets of any notable size only list the primes, not all the numbers. Because this project aims to build a binary classifier, I need data that is from both classes. (prime and composite). 

The second problem is that the data in the incorrect format for my purposes. Neural Networks require that the data in the input layer is between 0 and 1 for each input node. Therefore, it makes sense to feed in the binary representation for a given number, rather than its base-10 representation. 

The final (identified) problem is that the numbers have a limit. Sure, 50 million prime numbers would probably be good enough for most purposes, but if I end up needing more than that, then I want to have a technique that can generate arbitrarily large primes. 

With that said, I created a simple function in python that can determine if a number is prime or not. It loops through the possible factors of n, up to the square root of n. This is as high as one needs to check, because factors of a number come in pairs. One will be less than or equal to the square root of n, and the other will be greater than or equal to the square root of n.

```
import math

def isPrime(x):
    if x < 2:
        return(False)
    if x == 2:
        return(True)
    for i in range(2, int(math.ceil(math.sqrt(x))+1)):
        if x % i == 0:
            return(False)
    return(True)
```

## Data Cleaning

Taking a look at the dataset I created, I found that the vast majority (92%) of numbers were composite. This, as it turns out, means, that this is a binary classification problem with imbalanced classes. I decided to undersample the composite numbers prior to the training/testing split of the data. This prevented the model from being overly biased towards composite numbers. I wanted it to actually evaluate the prime-ness based on the number itself, rather than just making an educated guess because most numbers are composite.

I also considered removing numbers from the dataset that were obviously not prime. For example, any numbers that are even, or numbers that end in a 5. However, I decided against this, because part of what I want the network to learn is that these numbers are composite. (Which is not immediately obvious when given a binary representation.)

## Experimentation

With the dataset now formatted properly, a neural network was defined using keras. I tried a variety of models, but they all, surprisingly, had similar behavior. The number of layers didn’t seem to impact the model, and the number of nodes in each layer did not make much of a difference either. 

With that in mind, I ended up using a sequential model, with 64, 128, 32 nodes per layer in that order. There was also an output layer with just a single node. The middle layers utilized the ReLU activation, and the final output layer used the sigmoid function. Using ReLU in the output layer did seem to negatively impact the accuracy significantly, although the reason why is not immediately obvious. 

## Results

Regardless of most of how the hyperparameters were tuned, I found that the network quickly hit a ceiling as far as accuracy goes. That ceiling was at about 77% accuracy. However, there is something that I have a hard time explaining when it comes to model. After about 25 epochs the accuracy for both the validation and the testing dataset grew to about 84%.

![Prime Detection Results]("./images/PrimeDetection-ML-Results.png")

Something happened that was statistically significant at about 20 epochs, that was very odd. It was almost like it had learned to test for another divisibility rule, and the accuracy just improved!  This, however, is definitely not what I expected to see. All the literature that I found seemed to indicate that Neural Networks have a tendency to learn slowly, over time. However, in this case, it seems that there is something that happens when running for many epochs at a time.

## Conclusion

Neural Networks do show some promise for determining if a number is prime. It seems to quickly pick up on some basic divisibility rules, such as the even-ness of a number. (which is fairly easy to spot when viewing a number in its binary representation. However, this method does have some limitations associated with it. This neural network that I trained will only be valid at predicting numbers less than 2^20 – 1. This is a limitation on the network that I trained, but not on the method itself, because a new network could be trained with another node on the input layer.

However, I am cautiously optimistic about the use of neural networks in determining if a number is prime. If anybody wants to take this research further, I believe that augmenting the dataset to eliminate easy to spot composite numbers would be a useful thing to do. Perhaps also training a series of networks to test divisibility of certain factors could be used in parallel to test divisibility of a large amount of factors. (for example, one network tests divisibility by 2, another tests divisibility by 3, and so on). This would be similar to the research done by László and Schultz. It tests divisibility up to the square root, but that is largely similar to just testing divisibility directly.

## Source code

The source code for this project can be found [here](https://github.com/Brandonsams/ML-DetectingPrimes).