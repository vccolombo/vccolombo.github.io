---
title: "Campus Recruitment"
excerpt: "Data science project for the Campus Recruitment problem - How to choose the best MBA and land a great job?"
date: 2020-09-27T11:52:30-03:00
categories:
  - Data Science
  - Blog
tags:
  - Data Science
  - Machine Learning
  - Data Analysis
  - Classification
  - Regression
---

# Campus Recruitment - Data Science

## Introduction

One of the main concerns of students enrolled in all sorts of courses is whether they will find a job or not after finishing their studies.

In this report, I will analyze what factors influence the chances of finding good jobs after graduating from an MBA at an Indian university. Also, the results could be used by the institution to better select and guide their students during the course.

In the end, machine learning models will be created to try estimating if a student will find a job, and what wage it will receive.

### The Problem

Find what are the factors that lead to the best job opportunities for graduates from this institution.

### Questions

- What are the factors that lead to the person getting hired.
- What impacts the most on the salary.
- Can the institution predict if the person will get good results in the MBA before they are accepted?
  - The idea is that the institution would like to know beforehand if someone would fail to find a job after the MBA. The objective is to avoid accepting people that would not find a job and increase the employability rate of the institution.

### Benefits for the person:

The person will be able to know beforehand what to focus on to get the best results (placement/salary).

### Benefits for the institution:

The institution may be able to better select its students, reducing failures and placing its students in the best market positions. It might lead to higher renown for the institution.

Also, the institution may use the information to better prepare the course, focusing on the most important things to the market.

## Data

The [dataset](https://www.kaggle.com/benroshan/factors-affecting-campus-placement) consists of 215 rows representing students that completed an MBA course at an Indian University. The columns are as follow:

- **SerialNumber**: An increasing number which represents the row id.
- **Gender**: The gender of the student (M / F).
- **LowerSecondarySchollGrade%**: The average of grades in middle school (in %).
- **LowerSecondarySchollBoard**: Schools in India are separated in "Boards", depending on their region (Central / Others).
- **HigherSecondarySchollGrade%**: The average of grades in high school (in %).
- **HigherSecondarySchollBoard**: Same as middle school board, but for high school (Central / Others).
- **HigherSecondarySchollSpecialization**: The area of specialization during high school (Commerce / Science / Arts).
- **DegreeGrade%**: The average of grades in college (in %).
- **DegreeSpecialization**: The area of specialization in college (Sci&Tech / Comm&Mgmt / Others).
- **WorkExperience**: Whether the student had previous work experience (Yes / No).
- **EmployabilityTestGrade%**: A test that was made by the university to see if a student will find a job or not.
- **MBASpecialization**: The area of specialization in the MBA (Mkt&HR / Mkt&Fin).
- **MBAGrade%**: The average of grades in the MBA (in %).
- **PlacementStatus**: Whether the student finds a job or not after completing the MBA (Placed / Not Placed).
- **Salary**: The student salary after getting a job (only available if PlacementStatus == "Placed". NaN otherwise).

There are no missing values besides salary when the person is not employed. The dataset does not seem to contain incorrect data, but it has some outliers, like a person receiving a way bigger salary than the other students.

More information can be found in the [dataset page](https://www.kaggle.com/benroshan/factors-affecting-campus-placement).

## Analysis

Image 1 shows more students find a job after completing the MBA than those who don't, indicating that this is an unbalanced problem. However, I will not balance it. Instead, I chose to use a score to compare the models that take unbalance into account.

The problem with oversampling is that [it changes the data in a way that might affect the real-world representation of it](https://towardsdatascience.com/handling-imbalanced-datasets-in-machine-learning-7a0e84220f28). It could lead to bad results.

{% include figure image_path="/assets/images/data-science/campus-recruitment/PlacementStatus.png" alt="Placement Status distribution" caption="Image 1: Placement Status distribution." %}

### Employability Test

The first relationship of interest is the one between the Employability Test score (EmployabilityTestGrade%) and the person landing a job or not. Image 2 shows that some people who had a high score did not find a job, and a lot of people with low scores did get a position. However, the amount of people with high scores that were not employed is small compared to those who got a job.

{% include figure image_path="/assets/images/data-science/campus-recruitment/EmployabilityTest.png" alt="Relation between employability test grades and placement status" caption="Image 2: Relation between employability test grades and placement status." %}

However, there **is** a positive relation between salaries and the employability test. Image 3 shows an increase in salary for students that scored higher on the test.

{% include figure image_path="/assets/images/data-science/campus-recruitment/SalaryEmployabilityTest.png" alt="Relation between employability test grades and salary" caption="Image 3: Relation between employability test grades and salary." %}

### Gender

Analyzing the effects of gender on placement and salary, the conclusion was that women that came from men-dominated fields (areas with way more men, like Science and Commerce) tend to receive lower salaries. There is not enough data to conclude about women vs men when they had a background different from science or commerce, but the data available suggests women receive higher salaries in this scenario. In general, however, men receive higher salaries in both MBA specializations.

{% include figure image_path="/assets/images/data-science/campus-recruitment/DegreeSpecSalary.png" alt="Relation between college degree area and salary" caption="Image 4: Relation between college degree area and salary." %}

{% include figure image_path="/assets/images/data-science/campus-recruitment/MBASpecSalary.png" alt="Relation between MBA field of choice and salary" caption="Image 5: Relation between MBA field of choice and salary." %}

### Scores during the MBA course

The data suggests that having good grades impact positively on the salary a person will receive when employed.

{% include figure image_path="/assets/images/data-science/campus-recruitment/MBAGradesSalary.png" alt="Relation between grades during the MBA course and salary" caption="Image 6: Relation between grades during the MBA course and salary." %}

### Student's past

There is a lot of data about the students' past, like what board the school they attended is affiliated with, and their grades in all stages of education.

Interesting insights are that the boards have no impact on salary (this is shown in the correlation analysis too). However, the specialization the person chose during High school plays a role. Science and Commerce again show a salary boost when compared to Arts (see Image 7). Image 8 shows that Science degrees in college also result in better jobs.

{% include figure image_path="/assets/images/data-science/campus-recruitment/HighSchoolSpecSalary.png" alt="Relation between high school specialization area and salary" caption="Image 7: Relation between high school specialization area and salary." %}

{% include figure image_path="/assets/images/data-science/campus-recruitment/CollegeDegreeSalary.png" alt="Relation between college degree area and salary." caption="Image 8: Relation between college degree area and salary." %}

Also, there is a positive relation between grades in lower education and getting placed. However, I don't want to jump to conclusions in this one as it might indicate some underlying characteristics from the type of students that get good grades (as opposed to being something that is noticed by the employer). The analysis returned a correlation of up to 0.5 between school grades and placement status.

For the salary, though, there seems to be no evident relation between it and grades in previous education. The correlation returned a very small value (< 0.1).

### MBA field

The MBA area of choice is also a huge factor in salary, which can be seen in Image 9. An MBA in Finances leads to a higher salary than HR ones. Also, the Finances MBA results in a **WAY** better employability rate. Students that enrolled in an HR MBA struggled more to find a job.

{% include figure image_path="/assets/images/data-science/campus-recruitment/MBASpecSalaryNoGender.png" alt="Relation between MBA area and salary." caption="Image 9: Relation between MBA area and salary." %}

{% include figure image_path="/assets/images/data-science/campus-recruitment/MBASpecEmployability.png" alt="Relation between MBA area and employability." caption="Image 10: Relation between MBA area and employability." %}

### Previous work experience

This is another factor that greatly affects the salary and chance to find a job. Having worked before led to almost 10% higher salary on average. It also results in almost 50% better chances to be employed!

{% include figure image_path="/assets/images/data-science/campus-recruitment/WorkExperienceSalary.png" alt="Relation between having work experience and salary." caption="Image 11: Relation between having work experience and salary." %}

{% include figure image_path="/assets/images/data-science/campus-recruitment/WorkExperienceEmployability.png" alt="Relation between having work experience and employability." caption="Image 12: Relation between having work experience and employability." %}

## Conclusions

### Problem 1: What are the factors that lead to the person being hired:

The most important features for this problem are the MBA area of specialization, previous experience, and grades in general.

If the student's objective is to maximize the chance of finding a job after the MBA, it should select the Mkt&Fin course, find an internship to get experience (or work before the MBA). For younger students that are still on school and planing on the future, focusing on getting good grades might be beneficial too.

The Jupyter Notebook (link to the repository at the end of this text) contains a machine learning model to predict if an MBA student from this university will get a job using Random Forest. It scored 86.3% of precision.

### Problem 2: What are the major factors that affect the salary?

For this one, the important features were the MBA area, Employability Test grades, college degree specialization area, gender, and work experience.

Here again, choosing Mkt&Fin is a better option if the focus is on salary. However, school grades are not as relevant as in problem 1, as the employability test appears as a better indicator of the salary. Getting good grades in this test might indicate a better job. For students early in their education and thinking about what high school and college specializations to choose, the Science areas seem to be the most promising in terms of salary.

The amount of data available was not enough to create great models, but a Linear Regression was capable of returning good estimations.

### Problem 3: Can the institution predict if the person will be successful before they are accepted?

The best features here are the same as problem 1. So it is the same solution.

## Appendix

Check the source code in the [Github repository](https://github.com/vccolombo/campus-recruitment).
