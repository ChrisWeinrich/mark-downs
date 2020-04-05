# 8 Introducing a Separate Database for Queries

## Meet Scalability

Read is used the Most ! => Independent scaling

Commands is the master

1 master => more slaves for read

## Separation at the Data Level in the Real World

Indexed Views
Database Replication is inside SQL Server
elasticsearch

## Designing a Database for Queries

Different Schemes 

**Denormalize the Query Database => Flat Table**

## Creating a Database for Queries

```SQL
CREATE DATABASE [CqrsInPracticeReads]
go
USE [CqrsInPracticeReads]
GO
CREATE TABLE [dbo].[Student](
	[StudentID] [bigint] NOT NULL,
	[Name] [nvarchar](50) NOT NULL,
	[Email] [nvarchar](50) NOT NULL,
	[NumberOfEnrollments] [int] NOT NULL,
	[FirstCourseName] [nvarchar](50) NULL,
	[FirstCourseCredits] [int] NULL,
	[FirstCourseGrade] [int] NULL,
	[SecondCourseName] [nvarchar](50) NULL,
	[SecondCourseCredits] [int] NULL,
	[SecondCourseGrade] [int] NULL,
 CONSTRAINT [PK_Student] PRIMARY KEY CLUSTERED 
(
	[StudentID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]
```

## Scalability

**1 for Command as many as needed for Queries**

Also multiple read models are possible

## Caution

Much more complexity ! Wage the use against the Costs !

Single DB is often enough !