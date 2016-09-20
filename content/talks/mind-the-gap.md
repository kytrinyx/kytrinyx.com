+++
title = "Mind the Gap"
slug = "mind-the-gap"
date = "2016-07-12"
description = "How do you learn Go? First do the tour. Then read the language spec. Then write a lot of code. This works for a lot of people, but for some the gap between the tour and the spec is a vast chasm. This talk explores ways we might bridge this gap, and make Go accessible to a broader audience."
video = "https://www.youtube.com/embed/ClPIeuL9HnI"
+++

## Notes

When you want to learn Go, you will typically be told that the way to do so is:

* Do the Go Tour
* Read the Language Spec
* Write a lot of code

These are certainly not the only suggestions (nor the only available resources), but they are fairly typical. For some people the gap between the tour and the spec is a vast chasm.

## Motivation

There are at least three independent components that affect motivation.

* Subjective Value—if you believe that the goal is valuable, you will be more motivated.
* Expectancies for Success—if you believe that you can achieve the goal, then you're more likely to be motivated to try.
* Environment—a supportive environment is more motivating than a hostile one.

Expectancies for success can further be broken down:

* You need to have an actionable plan.
* You need to believe that following the plan will lead to success.
* You need to believe that you, personally, are capable of following the plan.

The plan that we typically lay out for people (tour, spec, write a lot of code) has issues that can greatly impact motivation for some people.

## Learning Styles

People learn very, very differently. A resource that one person finds helpful will leave another cold. One axis that demonstrates this is the "deep end" vs "baby steps" spectrum.

A person who favors the deep end approach will be unable to maintain interest in the face of baby steps, and they'll check out.

A person who favors the baby steps approach will panic if thrown in the deep end, and will learn very little.

## Suggestions for Improvement

I proposed three particular types of resource that could help bridge the gap between the tour and the spec.

The first is built on the idea that you can have a high degree of fluency at a low degree of proficiency.

> Fluency is what you can say when you're woken up in the middle of th enight with a flashlight in your face. &mdash;[Language Hunters][languagehunters]

In terms of language learning, the Language Hunters talk about <a href=“https://vimeo.com/6351731” class="web-links" target="blank"> four levels of proficiency</a>, using the example of going to a party.

* **Level 1**: You can express single, concrete ideas: _"Beer!"_, _"Good party!"_
* **Level 2**: You can communicate in full, simple sentences. _"Where is the party?"_
* **Level 3**: You can get a lot more descriptive. _"Last night at the party I fell into the punch bowl."_
* **Level 4**: You can have conversations about complex topics including social, economic, and political aspects. _"Should parties be illegal?"_

In terms of programming, this suggests that you could gain fluency in the syntax of the language, and familiarity with the standard library, idioms, and conventions, without having to tackle complex, real-world domains.

This is the idea behind Exercism, which provides numerous toy problems in about 30 different programming languages.

This however requires that the person doing the exercises receives feedback on their code.

The second suggestion is to specifically address gaps in people's knowledge of computer science topics by designing small problems that give people the opportunity to learn and practice them.

The third suggestion is to use working codebases in real-world domains as a scaffold for learning more complex topics in the language. The exercise would be to take the working code and either change it or add a feature.

## Resources

For the theoretical underpinnings of this, here are two great starting points:

* (book) [How Learning Works: 7 Research-based Principles for Smart Teaching][how-learning-works]
* (book) [Peak: Secrets from the New Science of Expertise][peak]

[languagehunters]: http://languagehunters.com
[how-learning-works]: https://www.amazon.com/How-Learning-Works-Research-Based-Principles/dp/0470484101
[peak]: https://www.amazon.com/Peak-Secrets-New-Science-Expertise/dp/0544456238
