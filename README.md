# SQL Project - Tech Tutoring

## About  

Explore my personal project using Stack Overflow's public database to help build a tech-focused tutoring agency. Completed in February 2024, this project includes queries to pinpoint potential subjects for tutoring, along with a comprehensive list of qualified tutors and students who could potentially benefit from our services.

## What is Stack Overflow?  

Stack Overflow is a popular online community and question-and-answer platform for programmers and developers. Users can post technical questions, seek assistance with coding issues, and share their expertise by answering questions. The platform covers a wide range of programming languages, frameworks, and development topics.

## Define the Goal  

The goal of this projct is to use Stack Overflow's database to help build a tech-focused tutoring agency.  
I'm specifically interested in the following areas:
1. Subjects to tutor
2. Tutors with expertise in these subjects
3. Students who need help in these subjects

## The Data

I will use Stack Overflow's database from the Stack Exchange network to complete this project. This database is used to store and manage the vast amount of content generated by Stack Overflow users and includes data dating back to 2008. The database contains nearly 30 tables and millions of rows. View the database schema [here](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede); I've also included a visual diagram of the schema below.  

Here are the tables and columns referenced in this project's queries:  
* **Posts**
    * Id - Id of post
    * PostTypeId - Used to identify post type; 1 = Question, 2 = Answer
    * AcceptedAnswerId - Id of the accepted answer, for questions 
    * ParentId - Id of the parent post, for answers
    * CreationDate - Date post was created
    * ViewCount - Number of views
    * OwnerUserId - User Id of post creator
    * Tags - Tags attached to post
* **Users**
    * Id - Id of user
    * Display Name - Display name of user
* **Tags**
    * Id - int identifier
    * TagName - char identifier
* **PostTags**
    * PostId - Id of post
    * TagId - Id of tag attached to post

## Which Subjects to Tutor

```sql
SELECT TOP 100
    Tags.TagName, 
    COUNT(*) AS TagCount,
    SUM(Posts.ViewCount) AS TotalViewCount
FROM Tags
JOIN PostTags ON Tags.Id = PostTags.TagId
JOIN Posts ON PostTags.PostId = Posts.Id
WHERE Posts.PostTypeId = 1 -- Questions only
    AND Posts.CreationDate >= DATEADD(year, -1, GETDATE()) -- Within the last year
GROUP BY Tags.TagName
ORDER BY TagCount DESC;
```

## Tutors with Expertise in these Subjects

```sql
DECLARE @Tag NVARCHAR(35) = LOWER('sql'); -- specify the tag

SELECT TOP 100
    Users.Id AS UserId,
    Users.DisplayName,
    COUNT(*) AS AcceptedAnswersCount  
FROM
    Users
JOIN
    Posts AS Answers ON Users.Id = Answers.OwnerUserId 
JOIN
    Posts AS Questions ON Answers.ParentId = Questions.Id
JOIN
    PostTags ON Questions.Id = PostTags.PostId
JOIN
    Tags ON PostTags.TagId = Tags.Id
WHERE
    Tags.TagName = @Tag
    AND Answers.PostTypeId = 2 -- Answers only
    AND Answers.Id = Questions.AcceptedAnswerId -- Accepted answers only
    AND Answers.CreationDate >= DATEADD(year, -2, GETDATE()) -- Within the last two years
GROUP BY
    Users.Id, Users.DisplayName
ORDER BY
    AcceptedAnswersCount DESC;
```

## Students who May Need Help in these Subjects

```sql
DECLARE @Tag NVARCHAR(35) = LOWER('sql'); -- specify the tag

SELECT TOP 100 Users.Id AS UserId, 
    Users.DisplayName, 
    COUNT(*) AS UnansweredQuestionsCount
FROM Users
JOIN Posts AS Questions ON Users.Id = Questions.OwnerUserId
JOIN PostTags ON Questions.Id = PostTags.PostId
JOIN Tags ON PostTags.TagId = Tags.Id
WHERE Tags.TagName = @Tag
    AND Questions.PostTypeId = 1 -- Questions only
    AND Questions.AcceptedAnswerId IS NULL -- Unanswered questions only
    AND Questions.CreationDate >= DATEADD(month, -6, GETDATE()) -- Within the last six months
GROUP BY Users.Id, Users.DisplayName
ORDER BY UnansweredQuestionsCount DESC
```
