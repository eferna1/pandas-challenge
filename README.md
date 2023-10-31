# pandas-challenge

#Dependencies & setup
import pandas as pd
from pathlib import Path

# %%
# Files to load
schools_data_to_load = Path("../Resources/schools_complete.csv")
students_data_to_load = Path("../Resources/students_complete.csv")

# Read data files and into Pandas DataFrames
school_data = pd.read_csv(schools_data_to_load)
student_data = pd.read_csv(students_data_to_load)

# Combine the school data into student data using school name
school_data_complete = pd.merge(student_data, school_data, how="left", on=["school_name", "school_name"])
school_data_complete.head()


# %% [markdown]
# # Disctrict Summary 

# %%
# Total no. of unique schools
school_count = school_data_complete["school_name"].nunique()
school_count

# %%
# Total no. of students
student_count = school_data_complete["student_name"].count()
student_count

# %%
# Total budget for school district
total_budget = sum(school_data_complete["budget"].unique())
total_budget

# %%
# Average math score
avg_math = school_data_complete["math_score"].mean()
avg_math

# %%
# Average reading score 
avg_reading = school_data_complete["reading_score"].mean()
avg_reading

# %%
# percentange of students who passed math 
#per_passing_math = (school_data_complete[school_data_complete["math_score"]>=70]["student_name"].count())/student_count
#per_passing_math
passing_math_count = school_data_complete[(school_data_complete["math_score"] >= 70)].count()["student_name"]
passing_math_percentage = passing_math_count / float(student_count) * 100
passing_math_percentage

# %%
# percentange of students who passed reading
passing_reading_count = school_data_complete[(school_data_complete["reading_score"] >= 70)].count()["student_name"]
passing_reading_percentage = passing_reading_count / float(student_count) * 100
passing_reading_percentage

# %%
# percentange of students who passed math and reading 
passing_math_reading_count = school_data_complete[
    (school_data_complete["math_score"] >= 70) & (school_data_complete["reading_score"] >= 70)
].count()["student_name"]
overall_passing_rate = passing_math_reading_count /  float(student_count) * 100
overall_passing_rate

# %%
# Creating dictionary to summarize the above metrics
district_summary = {"Total No. Schools" : [school_count], 
                    "Total No. Students" : [student_count], 
                    "Total Budget" : [total_budget], 
                    "Average Math Score" : [avg_math], 
                    "Average Reading Score" : [avg_reading], 
                    "Percent Passing Math": [passing_math_percentage], 
                    "Percent Passing Reading": [passing_reading_percentage], 
                    "Percent Overall Passing" : [overall_passing_rate]}

# Creating a dataframe to diplay the above metrics
district_summary = pd.DataFrame(district_summary)

# Format the dataframe
district_summary["Total No. Students"] = district_summary["Total No. Students"].map('{:,}'.format)
district_summary["Total Budget"] = district_summary["Total Budget"].map("${:,.2f}".format)

# Display district summary
district_summary

# %% [markdown]
# # School Summary

# %%
# Creating a groupby for school names
schools_grouped = school_data_complete.groupby(["school_name"])
print(schools_grouped)

# %%
# Use the code provided to select all of the school types
school_types = schools_grouped["type"].unique()
school_types

# %%
# Calculate the total student count per school
per_school_count = schools_grouped["student_name"].count()
per_school_count

# %%
# Calculate the total school budget and per capita spending per school
per_school_budget = schools_grouped["budget"].mean()
per_school_capita = per_school_budget / per_school_count
print(per_school_capita)

# %%
# Calculate the average test scores per school
# Average math score per school
per_school_math = schools_grouped["math_score"].mean()
per_school_math

# %%
# Average reading score per school
per_school_reading = schools_grouped["reading_score"].mean()
per_school_reading

# %%
# Calculate the number of students per school with math scores of 70 or higher
# Students passing math per school
students_passing_math = school_data_complete[school_data_complete["math_score"]>=70]
school_students_passing_math = students_passing_math.groupby(["school_name"]).size()
school_students_passing_math
per_school_passing_math = school_students_passing_math / per_school_count * 100
per_school_passing_math

# %%
# Calculate the number of students per school with reading scores of 70 or higher
students_passing_reading = school_data_complete[school_data_complete["reading_score"]>=70]
school_students_passing_reading= students_passing_reading.groupby(["school_name"]).size()
school_students_passing_reading
per_school_passing_reading = school_students_passing_reading / per_school_count * 100
per_school_passing_reading

# %%
# Use the provided code to calculate the number of students per school that passed both math and reading with scores of 70 or higher
students_passing_math_and_reading = school_data_complete[
    (school_data_complete["reading_score"] >= 70) & (school_data_complete["math_score"] >= 70)]
school_students_passing_math_and_reading = students_passing_math_and_reading.groupby(["school_name"]).size()
school_students_passing_math_and_reading
overall_passing_rate = school_students_passing_math_and_reading / per_school_count * 100
overall_passing_rate

# %%
# Create a DataFrame called `per_school_summary` with columns for the calculations above.
per_school_summary = pd.DataFrame ({"School Type" : school_types, 
                      "Total Students" : per_school_count, 
                      "Total School Budget" : per_school_budget,
                     "Per Student Budget" : per_school_capita,
                     "Average Math Score" : per_school_math,
                     "Average Reading Score" : per_school_reading,
                     "Percent Passing Math" : per_school_passing_math,
                     "Percent Passing Reading" : per_school_passing_reading,
                     "Percent Overall Passing" : overall_passing_rate})

# Formatting
per_school_summary["Total School Budget"] = per_school_summary["Total School Budget"].map("${:,.2f}".format)
per_school_summary["Per Student Budget"] = per_school_summary["Per Student Budget"].map("${:,.2f}".format)

school_types = school_data.set_index(["school_name"])["type"]

# Display the DataFrame
per_school_summary

# %% [markdown]
# # Highest-Performing Schools (by % Overall Passing)

# %%
# Sort the schools by `% Overall Passing` in descending order and display the top 5 rows.
top_schools = per_school_summary.sort_values("Percent Overall Passing", ascending=[False])
top_schools.head(5)

# %% [markdown]
# # Bottom Performing Schools (by % Overall Passing)

# %%
# Sort the schools by `% Overall Passing` in ascending order and display the top 5 rows.
bottom_schools = per_school_summary.sort_values("Percent Overall Passing")
bottom_schools.head(5)

# %% [markdown]
# # Math Scores by Grade

# %%
# Use the code provided to separate the data by grade
ninth_graders = school_data_complete[(school_data_complete["grade"] == "9th")]
tenth_graders = school_data_complete[(school_data_complete["grade"] == "10th")]
eleventh_graders = school_data_complete[(school_data_complete["grade"] == "11th")]
twelfth_graders = school_data_complete[(school_data_complete["grade"] == "12th")]

# Group by `school_name` and take the mean of the `math_score` column for each.
ninth_grade_math_scores = school_data_complete.loc[school_data_complete["grade"] == "9th"].groupby("school_name")["math_score"].mean()
tenth_grader_math_scores = school_data_complete.loc[school_data_complete["grade"] == "10th"].groupby("school_name")["math_score"].mean()
eleventh_grader_math_scores = school_data_complete.loc[school_data_complete["grade"] == "11th"].groupby("school_name")["math_score"].mean()
twelfth_grader_math_scores = school_data_complete.loc[school_data_complete["grade"] == "12th"].groupby("school_name")["math_score"].mean()

# Combine each of the scores above into single DataFrame called `math_scores_by_grade`
math_scores_by_grade = pd.DataFrame ({"9th" : ninth_grade_math_scores, 
                                      "10th" : tenth_grader_math_scores,
                                      "11th" : eleventh_grader_math_scores, 
                                      "12th" : twelfth_grader_math_scores})

# Minor data wrangling
math_scores_by_grade.index.name = None
math_scores_by_grade = math_scores_by_grade[["9th", "10th","11th", "12th"]]

# Display the DataFrame
math_scores_by_grade.style.format({"9th": "{:.6f}",
                                  "10th": "{:.6f}",
                                  "11th": "{:.6f}",
                                  "12th": "{:.6f}",})

# %% [markdown]
# # Reading Score by Grade

# %%
# Use the code provided to separate the data by grade
ninth_graders = school_data_complete[(school_data_complete["grade"] == "9th")]
tenth_graders = school_data_complete[(school_data_complete["grade"] == "10th")]
eleventh_graders = school_data_complete[(school_data_complete["grade"] == "11th")]
twelfth_graders = school_data_complete[(school_data_complete["grade"] == "12th")]

# Group by `school_name` and take the mean of the the `reading_score` column for each.
ninth_grade_reading_scores = school_data_complete.loc[school_data_complete["grade"] == "9th"].groupby("school_name")["reading_score"].mean()
tenth_grader_reading_scores = school_data_complete.loc[school_data_complete["grade"] == "10th"].groupby("school_name")["reading_score"].mean()
eleventh_grader_reading_scores = school_data_complete.loc[school_data_complete["grade"] == "11th"].groupby("school_name")["reading_score"].mean()
twelfth_grader_reading_scores = school_data_complete.loc[school_data_complete["grade"] == "12th"].groupby("school_name")["reading_score"].mean()

# Combine each of the scores above into single DataFrame called `reading_scores_by_grade`
reading_scores_by_grade = pd.DataFrame ({"9th" : ninth_grade_reading_scores, 
                                      "10th" : tenth_grader_reading_scores,
                                      "11th" : eleventh_grader_reading_scores, 
                                      "12th" : twelfth_grader_reading_scores})

# Minor data wrangling
reading_scores_by_grade = reading_scores_by_grade[["9th", "10th", "11th", "12th"]]
reading_scores_by_grade.index.name = None

# Display the DataFrame
reading_scores_by_grade.style.format({"9th": "{:.6f}",
                                  "10th": "{:.6f}",
                                  "11th": "{:.6f}",
                                  "12th": "{:.6f}",})

# %% [markdown]
# # Scores by School Spending

# %%
# Establish the bins 
spending_bins = [0, 585, 630, 645, 680]
spending_group = ["<$585", "$585-630", "$630-645", "$645-680"]

# %%
# Create a copy of the school summary since it has the "Per Student Budget" 
school_spending_df = per_school_summary.copy()

# %%
# Use `pd.cut` to categorize spending based on the bins.
school_spending_df["Spending Ranges (Per Student)"] = pd.cut(per_school_capita, 
                                                             spending_bins, 
                                                                labels=spending_group, 
                                                                 include_lowest=True)
school_spending_df

# %%
# calculate mean scores per spending range
spending_math_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Average Math Score"].mean()
spending_reading_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Average Reading Score"].mean()
spending_passing_math = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Percent Passing Math"].mean()
spending_passing_reading = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Percent Passing Reading"].mean()
overall_passing_spending = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Percent Overall Passing"].mean()


# %%
# Assemble into DataFrame
spending_summary = pd.DataFrame ({"Average Math Score" : spending_math_scores,
                                 "Average Reading Score": spending_reading_scores,
                                 "Percent Passing Math" : spending_passing_math,
                                 "Percent Passing Reading" : spending_passing_reading,
                                 "Percent Overall Passing" : overall_passing_spending})

# Display results
spending_summary

# %% [markdown]
# # Scores by School Size

# %%
# Establish the bins.
size_bins = [0, 1000, 2000, 5000]
school_size = ["Small (<1000)", "Medium (1000-2000)", "Large (2000-5000)"]

# %%
# Categorize the spending based on the bins
# Use `pd.cut` on the "Total Students" column of the `per_school_summary` DataFrame.

per_school_summary["School Size"] = pd.cut(per_school_count, size_bins, labels=school_size, include_lowest=True)
per_school_summary

# %%
# Calculate averages for the desired columns. 
size_math_scores = per_school_summary.groupby(["School Size"])["Average Math Score"].mean()
size_reading_scores = per_school_summary.groupby(["School Size"])["Average Reading Score"].mean()
size_passing_math = per_school_summary.groupby(["School Size"])["Percent Passing Math"].mean()
size_passing_reading = per_school_summary.groupby(["School Size"])["Percent Passing Reading"].mean()
size_overall_passing = per_school_summary.groupby(["School Size"])["Percent Overall Passing"].mean()

# %%
# Create a DataFrame called `size_summary` that breaks down school performance based on school size (small, medium, or large).
# Use the scores above to create a new DataFrame called `size_summary`
size_summary = pd.DataFrame ({"Average Math Score" : size_math_scores,
                              "Average Reading Score" : size_reading_scores,
                             "Percent Passing Math" : size_passing_math,
                             "Percent Passing Reading" : size_passing_reading,
                             "Percent Overall Passing" : size_overall_passing})

# Display results
size_summary

# %% [markdown]
# # Scores by School Type

# %%
average_math_score_by_type = per_school_summary.groupby(["School Type"])["Average Math Score"].mean()
average_reading_score_by_type = per_school_summary.groupby(["School Type"])["Average Reading Score"].mean()
average_percent_passing_math_by_type = per_school_summary.groupby(["School Type"])["Percent Passing Math"].mean()
average_percent_passing_reading_by_type = per_school_summary.groupby(["School Type"])["Percent Passing Reading"].mean()
average_percent_overall_passing_by_type = per_school_summary.groupby(["School Type"])["Percent Overall Passing"].mean()

# %%
average_math_score_by_type

# %%
# Assemble the new data by type into a DataFrame called `type_summary`
school_type_summary = pd.DataFrame({"Average Math Score" : average_math_score_by_type,
                             "Average Reading Score" : average_reading_score_by_type,
                             "Percent Passing Math" : average_percent_passing_math_by_type,
                             "Percent Passing Reading" : average_percent_passing_reading_by_type,
                             "Percent Overall Passing" : average_percent_overall_passing_by_type})

school_type_summary.index.name = "School Type"

# Display results
school_type_summary

# %% [markdown]
# # PyCitySchool Analysis

# %% [markdown]
# ###### Based on the School Summary, Test Scores, Budget and School Types: 
#     Students performed better in Reading over Math.
#     The highest performing, based on overall scores, were all Charter schools. 
#     Where as the lowest performing, based on overall scores, were all were District schools. 
#     Reading & Math scores were consistent through each grade per school. 
#     Schools that spent the least per student all had high overall passing scores.  
#     Also, the schools that spent the least per student were all Charter scools.
#     The majority of all large size shools had the lowest overall scores. 
#     All medium and small size shools, which are Charter schools, had overall passing scores. 
# 
# In conclusion Charter schools out performed District schools in both overall passing scores and budget spending. 
