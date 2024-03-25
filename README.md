![gophers](https://raw.githubusercontent.com/ashleymcnamara/gophers/master/GOPHER_AVATARS.jpg)

# ACS 4210: Patterns & Practices in Strongly Typed Languages

<!-- omit in toc -->
### Table of Contents

1. [Course Description](#course-description)
1. [Prerequisites](#prerequisites)
1. [Course Specifics](#course-specifics)
1. [Learning Objectives](#learning-objectives)
1. [Schedule](#schedule)
1. [Course Deliverables](#course-deliverables)
1. [Late Assignment Policy](#late-assignment-policy)
1. [Evaluation](#evaluation)
1. [Additional Resources](#additional-resources)

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

**Course Dates:** March 19 through May 11, 2024 _(7 weeks)_<br>
**Class Times:** 4:00pm - 6:45pm PST _(11 sessions, see below)_

### Holidays & Days Off

On the following days this term, our class will NOT be meeting on Zoom:

1. `Monday, April 01, 2024`: Campus Holiday
1. `Monday, April 22, 2024`: Lab Day for [Final Project](Project/MakeUtility.md)

### Class Sessions

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

## Course Deliverables

We will be using Gradescope this term, which allows us to provide fast and accurate feedback on your work. All assigned work will be submitted through Gradescope, and assignment and exam grades will be returned through Gradescope. As soon as grades are posted, you will be notified immediately so that you can log in and see your feedback. You may also submit regrade requests if you feel we have made a mistake.

Your Gradescope login is your Make School email, and your password can be changed at https://gradescope.com/reset_password. The same link can be used if you need to set your password for the first time.

*Assignments **must** be submitted to Gradescope by **11:59PM PST** on the date due.*

| 📚   Assignment                                      | 🔗   Criteria                                                                                                                                                                         | 📆   Due Date                                                        |
| :-------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------- |
| **[Tour of Go](https://tour.golang.org/welcome/1)** | Done in Class                                                                                                                                                                        | Jan 28, 2022  *(Friday)*                                            |
| **Static Site Generator**                           | [MVP](https://github.com/Make-School-Labs/makesite#mvp)  <br />[v1.1](https://github.com/Make-School-Labs/makesite#v1.1) / [v1.2](https://github.com/Make-School-Labs/makesite#v1.2) | <u>MVP</u>: TBD *(TBD)*<br /><u>v1.1</u> / <u>v1.2</u>: TBD *(TBD)* |
| **Web Scraper**                                     | [Requirements](https://github.com/Tech-at-DU/makescraper)                                                                                                                            | TBD (*TBD*)                                                         |
| **Blog Post**                                       | [Rubric](https://docs.google.com/document/d/1p5A_FvkVDJ783kYlCwBDpdGbY3G50UUJBmHH2umrzoU/edit?usp=sharing)                                                                           | March 11, 2022 *(Friday)*                                           |
| **MakeUtility Project <br>& Presentation**          | [Requirements](Project/MakeUtility.md)                                                                                                                                               | March 11, 2022 *(Friday)*                                           |

## Late Assignment Policy

- Late assignments that are submitted **more than 5 days (120 hours)** after the deadline will be given a **25% late penalty**.
- The **absolute last day** to submit any assignment will be **Friday, March 11 at 11:59 PM**.

If you require accommodations or have extenuating circumstances such as prolonged illness, please contact your instructor to request an extension.

## Evaluation

**To pass this course you must meet the following requirements**:

- Complete the [tutorial](#tutorial), deliverables, [final project](#final-project), and [final presentation](#final-presentation) as assigned in class and described in the sections below.
- Actively participate in class and abide by the attendance policy.
- Make up all classwork from all absences.

<!-- omit in toc -->
### Tutorial

Complete the [tutorial](https://tour.golang.org) assigned in class; assessed via graded warmup during week two.

<!-- omit in toc -->
### Blog Post

Demonstrate confidence writing and speaking about Go topics by writing a 500+ blog post on a language feature of your choice.

Your blog post must be accessible to the general public to earn credit; do not submit draft posts.

Your grade will be determined via the [Make School Blog Post Rubric](https://docs.google.com/document/d/1T1oqHFoRo0kl7mPUTFupmsoEkLYltKsVgtqyGKDaCgY/edit). **You must earn a score of 2.5 or higher to pass**.

<!-- omit in toc -->
### Final Project

Complete the final project according to the associated [project rubric](Project/MakeUtility.md).

<!-- omit in toc -->
### Final Presentation

The delivery of a live or pre-recorded presentation is required to pass this course. **Presentations will be delivered on our final day of class**.

Your **three to five minute presentation** should focus on the **experience you gained** and **lessons you learned** while implementing one of the three [Challenges](#challenges) in this course.

**Your final presentation will be evaluated based on the [Make School Presentation Rubric](https://docs.google.com/document/d/1WTLcZNyvRGYDz5L8Kr8a0ILbFAyr92u85paoqGFjxPg/edit). You must earn an average of 2.5 on the rubric to pass**.

## Additional Resources

https://threedots.tech/go-in-one-evening/

<!-- omit in toc -->
### Links

- [Gophercises](https://gophercises.com/): Real-world side projects with video tutorials!
- [TutorialEdge - Golang Repository](https://github.com/elliotforbes/tutorialedge-v2/tree/master/content/golang): Mini-tutorials to introduce and enhance your Golang knowledge.
- [YouTube - Todd Mcleod](https://www.youtube.com/user/toddmcleod/playlists): Videos to reinforce Golang concepts and techniques that we cover in class.
- [Echo Framework](https://echo.labstack.com/guide): Documentation for Echo, a high performance, extensible, minimalist Go web framework.
- [GORM](http://doc.gorm.io/#): The fantastic ORM library for Golang.
