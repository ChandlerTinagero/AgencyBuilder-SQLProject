# SQL Project - Tech Tutoring

## About  

Explore my personal project using Stack Overflow's public database to help build a tech-focused tutoring agency. Completed in February 2024, this project includes queries to pinpoint potential subjects for tutoring, along with a comprehensive list of qualified tutors and students.  

## What is Stack Overflow?  

Stack Overflow is a popular online community and question-and-answer platform for programmers and developers. Users can post technical questions, seek assistance with coding issues, and share their expertise by answering questions. The platform covers a wide range of programming languages, frameworks, and development topics.

## Define the Goal  

The goal of this project is to use Stack Overflow's database to help build a tech-focused tutoring agency.  
I'm specifically interested in using the database to inform the following decisions:
1. Determining which subject to teach
2. Identifying potential tutors with expertise in this subject
3. Identifying potential students who need help in this subject

## The Data

I will use Stack Overflow's database from the Stack Exchange network to complete this project. This database is used to store and manage the vast amount of content generated by Stack Overflow users and includes data dating back to 2008. The database contains nearly 30 tables and millions of rows. View the database schema and diagram [here](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede).  

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

## 1. Determining Which Subject to Teach

To decide which subject(s) my agency should tutor, I'll write a query to identify the ten subjects with the most questions. I'll also consider the total view count of these questions, since question views indicate that others are also interested in the subject. Finally, I'll filter the query for questions asked within the last year to ensure the results are relevant.  

**Query**  

```sql
SELECT TOP 10
    Tags.TagName AS Subject, 
    COUNT(*) AS QuestionCount,
    SUM(Questions.ViewCount) AS TotalViewCount
FROM Tags
    INNER JOIN PostTags ON Tags.Id = PostTags.TagId
    INNER JOIN Posts AS Questions ON PostTags.PostId = Questions.Id
WHERE Questions.PostTypeId = 1 -- Questions only
    AND Questions.CreationDate >= DATEADD(year, -1, GETDATE()) -- Within the last year
GROUP BY Tags.TagName
ORDER BY QuestionCount DESC; -- Sorting by subjects with the most questions
```

**Result**  
   
| Subject    | QuestionCount | TotalViewCount  |
|------------|---------------|-----------------|
| python     | 122943        | 25543353        |
| javascript | 80726         | 14027784        |
| reactjs    | 49821         | 13124532        |
| java       | 45381         | 8985404         |
| c#         | 41744         | 7313849         |
| android    | 33812         | 7309885         |
| html       | 31253         | 3532796         |
| r          | 28854         | 2599787         |
| flutter    | 27283         | 6021210         |
| node.js    | 26220         | 5828213         |

With tens of thousands of questions for each, any of the above subjects is a viable choice for my tutoring agency.  
I will choose Python as my agency's first tutoring subject.  

Now that I have a subject that's clearly in demand, I'll need to find qualified Python tutors to teach at my agency.  

## 2. Identifying Potential Tutors with Expertise in This Subject 

To find qualified Python tutors, I'll write a query to identify the ten users with the most accepted answers relating to Python. An answer becomes accepted once marked as such by the question asker. Therefore, users with many accepted answers are likely 1. knowledgeable and 2. skilled teachers. Like the previous query, I'll filter for answers within the last year to ensure relevance.  

**Query**

```sql
DECLARE @Subject NVARCHAR(35) = LOWER('python'); -- specify the subject

SELECT TOP 10
    Users.Id AS UserId,
    Users.DisplayName,
    COUNT(*) AS AcceptedAnswersCount  
FROM Users
    INNER JOIN Posts AS Answers ON Users.Id = Answers.OwnerUserId 
    INNER JOIN Posts AS Questions ON Answers.ParentId = Questions.Id
    INNER JOIN PostTags ON Questions.Id = PostTags.PostId
    INNER JOIN Tags ON PostTags.TagId = Tags.Id
WHERE Tags.TagName = @Subject
    AND Answers.PostTypeId = 2 -- Answers only
    AND Answers.Id = Questions.AcceptedAnswerId -- Accepted answers only
    AND Answers.CreationDate >= DATEADD(year, -1, GETDATE()) -- Within the last year
GROUP BY Users.Id, Users.DisplayName
ORDER BY AcceptedAnswersCount DESC; -- Sorting by users with the most accepted answers
```

**Result**

| UserId   | DisplayName                  | AcceptedAnswersCount |
|----------|------------------------------|----------------------|
| 16343464 | mozway                       | 1338                 |
| 10035985 | Andrej Kesely                | 696                  |
| 15239951 | Corralien                    | 601                  |
| 16120011 | Timeless                     | 429                  |
| 2901002  | jezrael                      | 336                  |
| 1491895  | Barmar                       | 224                  |
| 67579    | willeM_ Van Onsem            | 215                  |
| 5317403  | acw1668                      | 182                  |
| 12131013 | jared                        | 179                  |
| 13086128 | Goku - stands with Palestine | 166                  |

The users listed above have already spent hours helping others learn Python (for free!) and would likely be fantastic tutors at my agency. I can use their 'UserId' to message them on Stack Overflow. I could even include their 'AboutMe' and 'WebsiteUrl' to scan for email addresses, LinkedIn, or another form of contact.  

The only thing my agency is missing now is students!

## 3. Identifying Potential Students Who Need Help in This Subject

To find potential Python students, I'll write a query to identify the ten users with the most Python questions without an accepted answer. If a question does not have an accepted answer, it most likely means that the asker did not receive the help they sought. Therefore, users with many unanswered questions are prime candidates for tutoring. I'll filter for questions within the last six months.  

**Query**

```sql
DECLARE @Tag NVARCHAR(35) = LOWER('python'); -- specify the tag

SELECT TOP 10 Users.Id AS UserId, 
    Users.DisplayName, 
    COUNT(*) AS UnansweredQuestionsCount
FROM Users
    INNER JOIN Posts AS Questions ON Users.Id = Questions.OwnerUserId
    INNER JOIN PostTags ON Questions.Id = PostTags.PostId
    INNER JOIN Tags ON PostTags.TagId = Tags.Id
WHERE Tags.TagName = @Tag
    AND Questions.PostTypeId = 1 -- Questions only
    AND Questions.AcceptedAnswerId IS NULL -- Unanswered questions only
    AND Questions.CreationDate >= DATEADD(month, -6, GETDATE()) -- Within the last six months
GROUP BY Users.Id, Users.DisplayName
ORDER BY UnansweredQuestionsCount DESC -- Sorting by users with the most ananswered questions
```

**Result**

| UserId   | DisplayName               | UnansweredQuestionsCount  |
|----------|---------------------------|---------------------------|
| 893254   | FreelanceConsultant       | 31                        |
| 4451521  | KansaiRobot               | 27                        |
| 5924264  | roulette01                | 23                        |
| 726730   | Chris P                   | 20                        |
| 1422096  | Basj                      | 18                        |
| 2386113  | skm                       | 18                        |
| 23133316 | FrostDream                | 17                        |
| 22335013 | Jacques                   | 14                        |
| 22407544 | tthheemmaannii            | 14                        |
| 11748924 | Muhammad Ikhwan Perwira   | 14                        |

The users listed above have 14 to 30+ Python questions without accepted answers in the last six months alone. They're likely frustrated in their pursuit of answers and would be receptive to Python tutoring. I  could use their 'UserId' to message them on Stack Overflow, or include their 'AboutMe' and 'WebsiteUrl' to scan for another form of contact.

