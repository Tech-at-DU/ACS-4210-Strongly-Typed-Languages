![gophers](https://raw.githubusercontent.com/ashleymcnamara/gophers/master/GOPHER_AVATARS.jpg)

# ACS 4210: Patterns & Practices in Strongly Typed Languages

<!-- omit in toc -->
### Table of Contents

1. [Course Description](#%63%6F%75%72%73%65%2D%64%65%73%63%72%69%70%74%69%6F%6E)
1. [Prerequisites](#%70%72%65%72%65%71%75%69%73%69%74%65%73)
1. [Course Specifics](#%63%6F%75%72%73%65%2D%73%70%65%63%69%66%69%63%73)
1. [Learning Objectives](#%6C%65%61%72%6E%69%6E%67%2D%6F%62%6A%65%63%74%69%76%65%73)
1. [Schedule](#%73%63%68%65%64%75%6C%65)
   1. [Online Sessions](#%6F%6E%6C%69%6E%65%2D%73%65%73%73%69%6F%6E%73)
   1. [Deliverables](#%64%65%6C%69%76%65%72%61%62%6C%65%73)
1. [Evaluation](#%65%76%61%6C%75%61%74%69%6F%6E)
   1. [Tutorial](#%74%75%74%6F%72%69%61%6C)
   1. [Blog Post](#%62%6C%6F%67%2D%70%6F%73%74)
   1. [Final Project](#%66%69%6E%61%6C%2D%70%72%6F%6A%65%63%74)
   1. [Final Presentation](#%66%69%6E%61%6C%2D%70%72%65%73%65%6E%74%61%74%69%6F%6E)
1. [Late Assignment Policy](#%6C%61%74%65%2D%61%73%73%69%67%6E%6D%65%6E%74%2D%70%6F%6C%69%63%79)
1. [Additional Practice \& External Resources](#%61%64%64%69%74%69%6F%6E%61%6C%2D%70%72%61%63%74%69%63%65%2D%26%2D%65%78%74%65%72%6E%61%6C%2D%72%65%73%6F%75%72%63%65%73)


## Course Description

_In this course, students discover the value of strongly typed languages in server-side architectures, and dive deep into performant, concurrent programming paradigms present in Go. In studying Go, which is known for its ability to blend the expressive features of dynamic languages (Python, JavaScript) with the performance capabilities of compiled languages (C, C++), students will gain the syntactic diversity required in today's large-scale platform engineering pursuits. Throughout the course, students will learn and implement the design patterns and best practices that make Go a top choice at high-velocity startups like Lyft, Heroku, Docker, and Medium._

## Prerequisites

- [ACS 1220](https://make.sc/acs1220)

## Course Specifics

**Course Delivery**: online | 7 weeks<br>
**Course Credits**: 3 units | 37.5 Seat Hours | 75 Total Hours

## Learning Objectives

1. Design and implement command line interfaces, APIs, and bots in Go.
2. Identify and describe the architectures wherein the features of Golang could be best utilized.
3. Build data structures that support unmarshalling JSON retrieved from third-party APIs.
4. Apply Object Relational Mapping techniques to persist data to relational databases in Go.
5. Gain experience deploying APIs and bots to production.

## Schedule

**Course Dates:** Wednesday March 20 through Wednesday May 8, 2024 _(7 weeks)_<br>
**Class Times:** 4:00pm - 6:45pm PST _(11 sessions, see below)_

### Online Sessions

| Day | Class Topics                                                                    |
| :---: | ------------------------------------------------------------------------- |
| `01`  | 🆕 [Intro to Go / Tutorial Launch](Lessons/Tour.md)                       |
| `02` | 🆕 [Types & Static Languages](Lessons/Types.md)                 |
| `03`  | **[Static Site Generators](Lessons/SSGProject.md)**                       |
| `04`  | **[Files & Directories](Lessons/FilesDirectories.md)**                    |
| `05`  | **[Fast Functionality via 3rd Party Libraries](Lessons/3rdPartyLibs.md)** |
| `06`  | **[Scraping the Web](Lessons/WebScraping.md)** / **[Working With JSON](Lessons/JSON.md)** |
| `07`  | **[Final Project Intro](Project/MakeUtility.md)** / **[Pointers](Lessons/Pointers.md)**                               |
| `08`  | **[Concurrency & Goroutines](Lessons/Lesson07.md)**                       |
| `09`  | **[Benchmarking & Testing](Lessons/Lesson09.md)**                         |
| `10`  | **[Documentation & Deployments](Lessons/DocsDeploy.md)**                  |
| `11`  | [**Final Presentations**](Project/MakeUtility.md)                         |

### Deliverables

_ALL deliverables **must** be submitted to Gradescope by **11:59PM PST** on the date due._

📚 Assignment                                       | 🔗 Criteria                                                                                                                                                     | 📆 Due Date
:-------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------
**[Tour of Go](https://tour.golang.org/welcome/1)** | Done in Class                                                                                                                                                   | April 8, 2024 _(Monday)_
**Static Site Generator**                           | [MVP](https://github.com/Tech-at-DU/makesite#mvp)<br>[v1.1](https://github.com/Tech-at-DU/makesite#v1.1) / [v1.2](https://github.com/Tech-at-DU/makesite#v1.2) | <u>MVP</u>: April 3, 2024 (_Wednesday_)<br><u>v1.1</u> / <u>v1.2</u>: April 8, 2024 (_Monday_)
**Web Scraper**                                     | [Requirements](https://github.com/Tech-at-DU/makescraper)                                                                                                       | April 22, 2024 (_Monday_)
**Final Presentation**                              | [Requirements](Project/MakeUtility.md)                                                                                                                          | May 08, 2024 _(Wednesday)_
**Final Project**                                   | [Requirements](Project/MakeUtility.md)                                                                                                                          | May 12, 2024 _(Sunday)_
**Blog Post**                                       | [Rubric](https://docs.google.com/document/d/1p5A_FvkVDJ783kYlCwBDpdGbY3G50UUJBmHH2umrzoU/edit?usp=sharing)                                                      | May 12, 2024 _(Sunday)_

# Holidays & Offline Sessions

On the following days this term, our class will **NOT be meeting online** on Zoom:

1. `Monday, April 01, 2024`: Campus Holiday
2. `Monday, April 22, 2024`: Lab Day for [Final Project](Project/MakeUtility.md)

## Evaluation

We will be using Gradescope this term, which allows us to provide fast and accurate feedback on your work. All assigned work will be submitted through Gradescope, and assignment and exam grades will be returned through Gradescope. As soon as grades are posted, you will be notified immediately so that you can log in and see your feedback. You may also submit regrade requests if you feel we have made a mistake.

Your Gradescope login is your Dominican University email, and your password can be changed at <https://gradescope.com/reset_password>. The same link can be used if you need to set your password for the first time.

**To pass this course you must meet the following requirements**:

- Complete the [tutorial](#tutorial), [deliverables](#deliverables), [final project](#final-project), and [final presentation](#final-presentation) as assigned in class and described in the sections below.
- Make up all activities, classwork, and drills from all absences.
- Actively participate in class and abide by the attendance policy.

### Tutorial

Complete the [tutorial](https://tour.golang.org) assigned for homework on Day 01 and Day 02. Tutorial completion will be assessed via a series of drills, introduced by the instructor and completed in class. All drills must be turned in on [Gradescope](https://gradescope.com).

### Blog Post

Demonstrate confidence writing and speaking about Go topics by writing a 500+ blog post on a language feature of your choice.

Your blog post must be accessible to the general public to earn credit; do not submit draft posts.

Your grade will be determined via the [ACS Blog Post Rubric](). **You must earn a score of 2.5 or higher to pass**.

### Final Project

Complete the final project according to the associated [project rubric](Project/MakeUtility.md).

### Final Presentation

The delivery of a live or pre-recorded presentation is required to pass this course. **Presentations will be delivered on our final day of class**.

Your **three to five minute presentation** should focus on the **experience you gained** and **lessons you learned** while implementing one of the three [Challenges](#challenges) in this course.

**Your final presentation will be evaluated based on the ACS Presentation Rubric. You must earn an average of 2.5 on the rubric to pass**.

## Late Assignment Policy

The **absolute last day** to submit any assignment will be **Sunday, May 12 at 11:59 PM**.

If you require accommodations or have extenuating circumstances such as prolonged illness, please contact your instructor to request an extension.

## Additional Practice & External Resources

- [Gophercises](https://gophercises.com/): Real-world side projects with video tutorials!
- [TutorialEdge - Golang Repository](https://github.com/elliotforbes/tutorialedge-v2/tree/master/content/golang): Mini-tutorials to introduce and enhance your Golang knowledge.
- [YouTube - Todd Mcleod](https://www.youtube.com/user/toddmcleod/playlists): Videos to reinforce Golang concepts and techniques that we cover in class.
- [Echo Framework](https://echo.labstack.com/guide): Documentation for Echo, a high performance, extensible, minimalist Go web framework.
- [GORM](http://doc.gorm.io/#): The fantastic ORM library for Golang.
- <https://threedots.tech/go-in-one-evening/>
- <https://github.com/cblte/100-golang-exercises/blob/main/exercises_beginner.adoc>
