# Influncer's-Instagram-account-analysis-using-SQL

# Problem Statement:

Analyze Instagram activity data of a tech influencer to discover new trends, evaluate channel growth and suggest impactful content strategies to enhance engagement and audience reach.

# Task:

My task was to analyze the datasets and answer business questions, and present actionable insigts to the business stakeholders.

# Tool:

I have used MySQL Workbench for analysing the data.

# Business Questions to address:

1. How many unique post types are found in the 'fact_content' table?

   SELECT COUNT(DISTINCT(post_type)) AS Unique_Post_types
   FROM gdb0120.fact_content;


2. What are the highest and lowest recorded impressions for each post type?

   SELECT post_type, 
   MAX(impressions) AS Max_Impressions,
   MIN(impressions) AS Min_Impressions
   FROM gdb0120.fact_content
   GROUP BY post_type;


3. Filter all the posts that were published on a weekend in the month of March and April and export them to a separate csv file.

   SELECT c.* FROM gdb0120.fact_content c
   INNER JOIN gdb0120.dim_dates d ON 
   d.date = c.date
   WHERE d.month_name IN ('March','April') AND d.weekday_or_weekend = 'weekend';


4. Create a report to get the statistics for the account. The final output includes the following fields:
   • month_name
   • total_profile_visits
   • total_new_followers   
   
   SELECT d.month_name as Month_Name,
   SUM(a.profile_visits) AS Total_profile_visits,
   SUM(a.new_followers) as Total_new_followers
   FROM gdb0120.fact_account a
   INNER JOIN gdb0120.dim_dates d 
   ON a.date = d.date
   GROUP BY d.month_name;


5. Write a CTE that calculates the total number of 'likes’ for each 'post_category' during the month of 'July' and subsequently, arrange the 'post_category' values in descending order 
   according to their total likes.

   with cte1 as(
   SELECT c.post_category, SUM(c.likes) AS Likes
   FROM gdb0120.fact_content c
   INNER JOIN gdb0120.dim_dates d
   ON c.date = d.date 
   WHERE d.month_name = "July"
   GROUP BY c.post_category)

   SELECT cte1.* FROM cte1
   ORDER BY cte1.Likes Desc;


6. Create a report that displays the unique post_category names alongside their respective counts for each month. The output should have three columns:
   • month_name
   • post_category_names
   • post_category_count
   Example:
   • 'April', 'Earphone,Laptop,Mobile,Other Gadgets,Smartwatch', '5'
   • 'February', 'Earphone,Laptop,Mobile,Smartwatch', '4'
   

   SELECT d.month_name AS Month_Name,
   GROUP_CONCAT(DISTINCT c.post_category) AS post_category_name,
   LENGTH(GROUP_CONCAT(DISTINCT c.post_category)) - 
   LENGTH(REPLACE(GROUP_CONCAT(DISTINCT c.post_category), ",", "")) +1 AS post_category_count
   FROM gdb0120.fact_content c
   INNER JOIN gdb0120.dim_dates d ON
   d.date = c.date
   GROUP BY Month_Name
   ORDER BY post_category_count DESC;


7. What is the percentage breakdown of total reach by post type? The final output includes the following fields:
  • post_type
  • total_reach
  • reach_percentage

  With cte1 as(
  SELECT post_type, SUM(reach) as total_reach
  FROM gdb0120.fact_content
  GROUP BY post_type)

  SELECT cte1.*, 
  ROUND((cte1.total_reach / SUM(cte1.total_reach) OVER()) *100 ,2)as reach_percentage
  FROM cte1
  ORDER BY reach_percentage DESC;


8. Create a report that includes the quarter, total comments, and total saves recorded for each post category. Assign the following quarter groupings: (January, February, March) → “Q1”
   (April, May, June) → “Q2”
   (July, August, September) → “Q3”
   The final output columns should consist of:
   • post_category
   • quarter
   • total_comments
   • total_saves


   SELECT c.post_category, 
   Case 
      when d.month_name IN ("January", "February", "March") then "Q1"
      when d.month_name IN ("April", "May", "June") then "Q2"
      when d.month_name IN ("July", "August", "September") then "Q3"
   end as Quarter,
   SUM(c.comments) as total_comments,
   SUM(c.saves) as total_saves
   FROM gdb0120.fact_content c
   INNER JOIN gdb0120.dim_dates d
   ON c.date = d.date
   GROUP BY c.post_category, Quarter
   ORDER BY c.post_category, Quarter;


9. List the top three dates in each month with the highest number of new followers. The final output should include the following columns:
   • month
   • date
   • new_followers   


   with cte1 as(
   SELECT d.month_name, d.date, SUM(a.new_followers) as new_followers,
   Dense_rank() OVER(PARTITION BY d.month_name ORDER BY SUM(a.new_followers) DESC) as Rnk 
   FROM gdb0120.fact_account a
   INNER JOIN gdb0120.dim_dates d 
   ON a.date = d.date
   GROUP BY d.month_name, d.date)

   SELECT cte1.month_name, cte1.date, cte1.new_followers
   FROM cte1 
   WHERE Rnk <= 3
   ORDER BY cte1.date ASC; 
  

10. Create a stored procedure that takes the 'Week_no' as input and generates a report displaying the total shares for each 'Post_type'. The output of the procedure should consist of 
    two columns:
    • post_type
    • total_shares 


   CREATE PROCEDURE weekly_post_shares_report(Week_no VARCHAR(255))
   SELECT c.post_type, SUM(c.shares) as total_shares
   FROM gdb0120.fact_content c 
   INNER JOIN gdb0120.dim_dates d ON 
   c.date = d.date
   WHERE d.week_no = Week_no
   GROUP BY c.post_type;

  CALL weekly_post_shares_report("W3");
   
