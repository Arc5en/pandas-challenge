# pandas-challenge
# The main purpose of this code is to analyze school test results in a school district. This is the readme version with
# detailed annotation explaining the code from a personal perspective. 
# Most of this code was inspired from TA Drew's speedrun.
# Please refer to the comments below for an extended written report on the analysis.


# In[ ]:


#Initial Steps to Open CSV files in Python/Pandas & Preparation for Analysis
# Dependencies and Setup
import pandas as pd
from pathlib import Path

# File to Load (Remember to Change These)
school_data_to_load = Path("Resources/schools_complete.csv")
student_data_to_load = Path("Resources/students_complete.csv")

# Read School and Student Data File and store into Pandas DataFrames
school_data = pd.read_csv(school_data_to_load)
student_data = pd.read_csv(student_data_to_load)

# Combine the data into a single dataset.  
school_data_complete = pd.merge(student_data, school_data, how="left", on=["school_name", "school_name"])
school_data_complete.head()


# ## District Summary

# In[ ]:


#Overview of All Students in School District


# In[ ]:


# Calculate the total number of unique schools: Uses school_name column and counts distinct values in column
school_count = len(school_data_complete["school_name"].unique())
school_count


# In[ ]:


# Calculate the total number of students, based on distinct student ID. Unique or count founction works here
student_count = len(school_data_complete["Student ID"].unique())
student_count


# In[ ]:


# Calculate the total budget: does not use school_data_complete data frame since budget is from school data, not student data
total_budget = school_data["budget"].sum()
total_budget


# In[ ]:


# Calculate the average (mean) math score
average_math_score = school_data_complete["math_score"].mean()
average_math_score


# In[ ]:


# Calculate the average (mean) reading score
average_reading_score = school_data_complete["reading_score"].mean()
average_reading_score


# In[ ]:


# Use the following to calculate the percentage of students who passed math (math scores greather than or equal to 70)
passing_math_count = school_data_complete[(school_data_complete["math_score"] >= 70)].count()["student_name"]
passing_math_percentage = passing_math_count / float(student_count) * 100
passing_math_percentage


# In[ ]:


# Calculate the percentage of students who passed reading (hint: look at how the math percentage was calculated)  
passing_reading_count = school_data_complete[(school_data_complete["reading_score"] >= 70)].count()["student_name"]
passing_reading_percentage = passing_reading_count / float(student_count) * 100
passing_reading_percentage


# In[ ]:


# Use the following to calculate the percentage of students that passed math and reading
passing_math_reading_count = school_data_complete[
    (school_data_complete["math_score"] >= 70) & (school_data_complete["reading_score"] >= 70)
].count()["student_name"]
overall_passing_rate = passing_math_reading_count /  float(student_count) * 100
overall_passing_rate


# In[ ]:


# Create a high-level snapshot of the district's key metrics in a DataFrame
district_summary = pd.DataFrame(
    {
        "Total Schools": [school_count],
        "Total Students": [student_count],
        "Total Budget": [total_budget],
        "Average Math Score": [average_math_score],
        "Average Reading Score": [average_reading_score],
        "% Passing Math": [passing_math_percentage],
        "% Passing Reading": [passing_reading_percentage],
        "Overall Passing": [overall_passing_rate]
        
    }

)

# Formatting
district_summary["Total Students"] = district_summary["Total Students"].map("{:,}".format)
district_summary["Total Budget"] = district_summary["Total Budget"].map("${:,.2f}".format)

# Display the DataFrame
district_summary


# ## School Summary

# In[ ]:


# Overview of Stats of Students sorted by enrolled school


# In[ ]:


# Use the code provided to select all of the school types
school_types = school_data.set_index(["school_name"])["type"]
# school_types


# In[ ]:


# Calculate the total student count per school
per_school_counts = school_data_complete["school_name"].value_counts()
# per_school_counts


# In[ ]:


# Calculate the total school budget and per capita spending per school
per_school_budget = school_data_complete.groupby(["school_name"]).mean()["budget"]
per_school_capita = per_school_budget / per_school_counts


# In[ ]:


# Calculate the average test scores per school
per_school_math = school_data_complete.groupby(["school_name"]).mean()["math_score"]
per_school_reading = school_data_complete.groupby(["school_name"]).mean()["reading_score"]


# In[ ]:


# Calculate the number of schools with math scores of 70 or higher


# In[ ]:


# Calculate the number of students per school with math scores of 70 or higher
students_passing_math = school_data_complete[
    (school_data_complete["math_score"] >= 70)
]
school_students_passing_math = students_passing_math.groupby(["school_name"]).size()


# In[ ]:


# Calculate the number of students per school with reading scores of 70 or higher
students_passing_reading = school_data_complete[
    (school_data_complete["reading_score"] >= 70)
]
school_students_passing_reading = students_passing_reading.groupby(["school_name"]).size()


# In[ ]:


# Use the provided code to calculate the number of students per school that passed both math and reading with scores of 70 or higher
students_passing_math_and_reading = school_data_complete[
    (school_data_complete["reading_score"] >= 70) & (school_data_complete["math_score"] >= 70)
]
school_students_passing_math_and_reading = students_passing_math_and_reading.groupby(["school_name"]).size()


# In[ ]:


# Use the provided code to calculate the passing rates
per_school_passing_math = school_students_passing_math / per_school_counts * 100
per_school_passing_reading = school_students_passing_reading / per_school_counts * 100
overall_passing_rate = school_students_passing_math_and_reading / per_school_counts * 100


# In[ ]:


# Create a DataFrame called `per_school_summary` with columns for the calculations above.
per_school_summary = pd.DataFrame(
    {
        "School Type": school_types,
        "Total Students": per_school_counts,
        "Total School Budget": per_school_budget,
        "Per Student Budget": per_school_capita,
        "Average Math Score": per_school_math,
        "Average Reading Score": per_school_reading,
        "% Passing Math": per_school_passing_math,
        "% Passing Reading": per_school_passing_reading,
        "% Overall Passing": overall_passing_rate
    }
)

# Formatting
per_school_summary["Total School Budget"] = per_school_summary["Total School Budget"].map("${:,.2f}".format)
per_school_summary["Per Student Budget"] = per_school_summary["Per Student Budget"].map("${:,.2f}".format)

# Display the DataFrame
per_school_summary


# ## Highest-Performing Schools (by % Overall Passing)

# In[ ]:


# Snapshot of schools with highest passing rates. Note the corresponding table lists schools from #1 to #5 highest passing rates.
# Sort the schools by `% Overall Passing` in descending order and display the top 5 rows.
top_schools = per_school_summary.sort_values("% Overall Passing", ascending = False)
top_schools.head(5)


# ## Bottom Performing Schools (By % Overall Passing)

# In[ ]:


# Snapshot of schools with lowest passing rates. Note the table lists schools from school with lowest passing rate at the top, where subsequent rows will have higher passing rates.
# Sort the schools by `% Overall Passing` in ascending order and display the top 5 rows.
bottom_schools = per_school_summary.sort_values("% Overall Passing", ascending = True)
bottom_schools.head(5)


# ## Math Scores by Grade

# In[ ]:


# Exploring the relationship between grade level and math scores given for each school
# Use the code provided to separate the data by grade
ninth_graders = school_data_complete[(school_data_complete["grade"] == "9th")]
tenth_graders = school_data_complete[(school_data_complete["grade"] == "10th")]
eleventh_graders = school_data_complete[(school_data_complete["grade"] == "11th")]
twelfth_graders = school_data_complete[(school_data_complete["grade"] == "12th")]

# Group by `school_name` and take the mean of the `math_score` column for each.
ninth_graders_scores = ninth_graders.groupby(["school_name"]).mean()
tenth_graders_scores = tenth_graders.groupby(["school_name"]).mean()
eleventh_graders_scores = eleventh_graders.groupby(["school_name"]).mean()
twelfth_graders_scores = twelfth_graders.groupby(["school_name"]).mean()

ninth_graders_math_scores = ninth_graders_scores["math_score"]
tenth_graders_math_scores = tenth_graders_scores["math_score"]
eleventh_graders_math_scores = eleventh_graders_scores["math_score"]
twelfth_graders_math_scores = twelfth_graders_scores["math_score"]

# Combine each of the scores above into single DataFrame called `math_scores_by_grade`
math_scores_by_grade = pd.DataFrame(
    {
        "9th": ninth_graders_math_scores,
        "10th": tenth_graders_math_scores,
        "11th": eleventh_graders_math_scores,
        "12th": twelfth_graders_math_scores
    }
)

# Minor data wrangling
math_scores_by_grade.index.name = None

# Display the DataFrame
math_scores_by_grade
# Overall, the math scores support the conclusions drawn from school rankings of highest/lowest overall passing rates.


# ## Reading Score by Grade 

# In[ ]:


# Exploring the relationship between grade level and reading scores given for each school
# Use the code provided to separate the data by grade
ninth_graders = school_data_complete[(school_data_complete["grade"] == "9th")]
tenth_graders = school_data_complete[(school_data_complete["grade"] == "10th")]
eleventh_graders = school_data_complete[(school_data_complete["grade"] == "11th")]
twelfth_graders = school_data_complete[(school_data_complete["grade"] == "12th")]

# Group by `school_name` and take the mean of the the `reading_score` column for each.
ninth_graders_reading_scores = ninth_graders_scores["reading_score"]
tenth_graders_reading_scores = tenth_graders_scores["reading_score"]
eleventh_graders_reading_scores = eleventh_graders_scores["reading_score"]
twelfth_graders_reading_scores = twelfth_graders_scores["reading_score"]

# Combine each of the scores above into single DataFrame called `reading_scores_by_grade`
reading_scores_by_grade = pd.DataFrame(
    {
        "9th": ninth_graders_reading_scores,
        "10th": tenth_graders_reading_scores,
        "11th": eleventh_graders_reading_scores,
        "12th": twelfth_graders_reading_scores
    }
)

# Minor data wrangling
reading_scores_by_grade = reading_scores_by_grade[["9th", "10th", "11th", "12th"]]
reading_scores_by_grade.index.name = None

# Display the DataFrame
reading_scores_by_grade
# Overall, the reading scores also support the conclusions drawn from school rankings of highest/lowest overall passing rates.


# ## Scores by School Spending

# In[ ]:


# Exploring the relationship between school spending per student and passing rates for each school.
# Establish the bins 
spending_bins = [0, 585, 630, 645, 680]
labels = ["<$585", "$585-630", "$630-645", "$645-680"]


# In[ ]:


# Create a copy of the school summary since it has the "Per Student Budget" 
school_spending_df = per_school_summary.copy()


# In[ ]:


# Use `pd.cut` to categorize spending based on the bins.
school_spending_df["Spending Ranges (Per Student)"] = pd.cut(per_school_capita, bins=spending_bins, labels=labels)
school_spending_df


# In[ ]:


#  Calculate averages for the desired columns. 
spending_math_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Average Math Score"].mean()
spending_reading_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Average Reading Score"].mean()
spending_passing_math = school_spending_df.groupby(["Spending Ranges (Per Student)"])["% Passing Math"].mean()
spending_passing_reading = school_spending_df.groupby(["Spending Ranges (Per Student)"])["% Passing Reading"].mean()
overall_passing_spending = school_spending_df.groupby(["Spending Ranges (Per Student)"])["% Overall Passing"].mean()


# In[ ]:


# Assemble into DataFrame
spending_summary = pd.DataFrame(
    {
        "Average Math Score": spending_math_scores,
        "Average Reading Score": spending_reading_scores,
        "% Passing Math": spending_passing_math,
        "% Passing Reading": spending_passing_reading,
        "% Overall Passing": overall_passing_spending
    }
)


# Display results
spending_summary
# School with higher overall passing rates also spend less on each student (mostly due to lower budget, and noticeably has lower student count).


# ## Scores by School Size

# In[ ]:


# Exploring the relationship between student population size and passing rates given for each school
# Establish the bins.
size_bins = [0, 1000, 2000, 5000]
labels = ["Small (<1000)", "Medium (1000-2000)", "Large (2000-5000)"]


# In[ ]:


# Categorize the spending based on the bins
# Use `pd.cut` on the "Total Students" column of the `per_school_summary` DataFrame.

per_school_summary["School Size"] = pd.cut(per_school_summary["Total Students"], bins=size_bins, labels=labels)
per_school_summary


# In[ ]:


# Calculate averages for the desired columns. 
size_math_scores = per_school_summary.groupby(["School Size"])["Average Math Score"].mean()
size_reading_scores = per_school_summary.groupby(["School Size"])["Average Reading Score"].mean()
size_passing_math = per_school_summary.groupby(["School Size"])["% Passing Math"].mean()
size_passing_reading = per_school_summary.groupby(["School Size"])["% Passing Reading"].mean()
size_overall_passing = per_school_summary.groupby(["School Size"])["% Overall Passing"].mean()


# In[ ]:


# Create a DataFrame called `size_summary` that breaks down school performance based on school size (small, medium, or large).
# Use the scores above to create a new DataFrame called `size_summary`
size_summary = pd.DataFrame(
    {
        "Average Math Score": size_math_scores,
        "Average Reading Score": size_reading_scores,
        "% Passing Math": size_passing_math,
        "% Passing Reading": size_passing_reading,
        "% Overall Passing": size_overall_passing
    }
)

# Display results
size_summary
# This table follows up on the relationship of higher passing rates with lower student populations.
# Understandably so, as teacher to student ratios are more likely to be smaller in schools with small student populations.


# ## Scores by School Type

# In[ ]:


# Exploring the relationship between charter vs district school and test results.
# Group the per_school_summary DataFrame by "School Type" and average the results.
average_math_score_by_type = per_school_summary.groupby(["School Type"])["Average Math Score"].mean()
average_reading_score_by_type = per_school_summary.groupby(["School Type"])["Average Reading Score"].mean()
average_percent_passing_math_by_type = per_school_summary.groupby(["School Type"])["% Passing Math"].mean()
average_percent_passing_reading_by_type = per_school_summary.groupby(["School Type"])["% Passing Reading"].mean()
average_percent_overall_passing_by_type = per_school_summary.groupby(["School Type"])["% Overall Passing"].mean()


# In[ ]:


# Assemble the new data by type into a DataFrame called `type_summary`
type_summary = pd.DataFrame(
    {
        "Average Math Score": average_math_score_by_type,
        "Average Reading Score": average_reading_score_by_type,
        "% Passing Math": average_percent_passing_math_by_type,
        "% Passing Reading": average_percent_passing_reading_by_type,
        "% Overall Passing": average_percent_overall_passing_by_type
    }
)

# Display results
type_summary
# There is a significant gap between passing rates of district school vs charter schools.
# While the differences in averages are smaller, the difference in passing rates is a consistently noticeable trend in each relationship explored.


# In[ ]:


# Written Report
# While there was a noticeable gap between the averages of test scores of top performing school and bottom performing schools,
# the gap between both groups in passing rates was far more significant.
# One noticeable common trait top performers had were their classification as charter schools, whereas bottom performers
# were more likely to be district schools.
# This may also explain why schools with smaller student population sizes had higher passing rates 
# compared to schools with larger student populations. Smaller student bodies are more likely to have smaller class sizes,
# which are associated with higher quality environments for teaching.
# As an additional note, top performers also had lower budgets, meaning less money was spent on per student.
# This point may have potential follow up to see where exactly the money is distributed in each school's budget.
# The only point which did not exhibit as much of a significant difference were grades and test scores for each student,
# as each school had little difference in test scores between each grade. Perhaps exploring their passing rates,
# may provide some insight into their performances on a deeper level.
