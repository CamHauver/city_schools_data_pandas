# # PyCity Schools Analysis
# 
# - Analysis:
#   
# --- Combining the data from both CSV files (schools_complete and students_complete) results in a dataset generally summarized as follows (see district_summary DataFrame):
#     1) Total Schools: 15
#     2) Total Students: 39,170
#     3) Total Budget: $24,649,428.00
#     4) Average Math Score: 78.99
#     5) Average Reading Score: 81.88%
#     6) Passing Math Percentage: 74.98%
#     7) Passing Reading Percentage: 85.81%
#     8) Overall Passing Rate: 65.17%
#     
# --- Listed below are some more insights found by analyzing the data further:
#     1) The highest performing schools, based on Overall Passing Rate, are all Charter schools (#1 = Cabrera High School, #2 = Thomas High School, #3 = Griffin High School, #4 = Wilson High School, #5 = Pena High School) (See top_schools.head(5) Dataframe)
#     2) The lowest performing schools, based on Overall Passing Rate, are all District schools (#15 = Rodriguez High School, #14 = Figueroa High School, #13 = Huang High School, #12 = Hernandez High School, #11 = Johnson High School) (See bottom_schools.head(5) Dataframe)
#     3) Pooling data by grade (9th, 10th, 11th, and 12th) for math and reading scores does not reveal any obvious trends. Further analysis would be required to draw conclusions about how math and reading scores compare between grades (See math_scores_by_grade and reading_scores_by_grade Dataframes)
#     4) Average math and reading passing rates trend higher with less spending per student (See spending_summary Dataframe)
#     5) Average math and reading passing rates trend higher with small and medium schools. Large schools show dramatically reduced passing rates, more so in math than reading. (See size_summary Dataframe)
#     6) Charter schools perform better than District schools, with higher math and reading scores and passing rates. In particular, the average percentage of students passing math differs drastically between school type, with Charter schools at 93.6% and District schools at 66.5%. There's a dramatic difference between school type and overall passing rate, with Charter schools at 90.4% and District schools at 53.7% (See type_summary Dataframe)

# In[1]:


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

# In[2]:


# Calculate the total number of unique schools
school_count = len(school_data_complete["school_name"].unique())
school_count


# In[3]:


# Calculate the total number of students
student_count = school_data_complete["Student ID"].count()
student_count


# In[4]:


# Calculate the total budget
total_budget = school_data["budget"].sum()
total_budget


# In[5]:


# Calculate the average (mean) math score
average_math_score = school_data_complete["math_score"].mean()
average_math_score


# In[6]:


# Calculate the average (mean) reading score
average_reading_score = school_data_complete["reading_score"].mean()
average_reading_score


# In[7]:


# Use the following to calculate the percentage of students who passed math (math scores greather than or equal to 70)
passing_math_count = school_data_complete[(school_data_complete["math_score"] >= 70)].count()["student_name"]
passing_math_percentage = passing_math_count / float(student_count) * 100
passing_math_percentage


# In[8]:


# Calculate the percentage of students who passed reading (hint: look at how the math percentage was calculated)
passing_reading_count = school_data_complete[(school_data_complete["reading_score"] >= 70)].count()["student_name"]
passing_reading_percentage = passing_reading_count / float(student_count) * 100
passing_reading_percentage


# In[9]:


# Use the following to calculate the percentage of students that passed math and reading
passing_math_reading_count = school_data_complete[
    (school_data_complete["math_score"] >= 70) & (school_data_complete["reading_score"] >= 70)
].count()["student_name"]
overall_passing_rate = passing_math_reading_count /  float(student_count) * 100
overall_passing_rate


# In[10]:


# Create a high-level snapshot of the district's key metrics in a DataFrame
district_summary = pd.DataFrame({
    "Total Schools": [school_count],
    "Total Students": [student_count],
    "Total Budget": [total_budget],
    "Average Math Score": [average_math_score],
    "Average Reading Score": [average_reading_score],
    "Passing Math Percentage": [passing_math_percentage],
    "Passing Reading Percentage": [passing_reading_percentage],
    "Overall Passing Rate": [overall_passing_rate]
})

# Formatting
district_summary["Total Students"] = district_summary["Total Students"].map("{:,}".format)
district_summary["Total Budget"] = district_summary["Total Budget"].map("${:,.2f}".format)
district_summary["Average Math Score"] = district_summary["Average Math Score"].map("{:.2f}".format)
district_summary["Average Reading Score"] = district_summary["Average Reading Score"].map("{:.2f}".format)
district_summary["Passing Math Percentage"] = district_summary["Passing Math Percentage"].map("{:.2f}".format)
district_summary["Passing Reading Percentage"] = district_summary["Passing Reading Percentage"].map("{:.2f}".format)
district_summary["Overall Passing Rate"] = district_summary["Overall Passing Rate"].map("{:.2f}".format)

# Display the DataFrame
district_summary


# ## School Summary

# In[11]:


# Use the code provided to select the type per school from school_data
school_types = school_data.set_index(["school_name"])["type"]
school_types


# In[12]:


# Calculate the total student count per school from school_data
per_school_counts = school_data_complete["school_name"].value_counts()
per_school_counts


# In[13]:


# Calculate the total school budget and per capita spending per school from school_data
per_school_budget = school_data_complete.groupby(["school_name"]).mean(numeric_only=True)["budget"]

per_school_capita = per_school_budget / per_school_counts
per_school_capita


# In[14]:


# Calculate the average test scores per school from school_data_complete
per_school_math = school_data_complete.groupby(["school_name"]).mean(numeric_only=True)["math_score"]

per_school_reading = school_data_complete.groupby(["school_name"]).mean(numeric_only=True)["reading_score"]


# In[15]:


# Calculate the number of students per school with math scores of 70 or higher from school_data_complete
students_passing_math = school_data_complete[school_data_complete["math_score"] >= 70]
school_students_passing_math = students_passing_math.groupby(["school_name"]).count()["student_name"]


# In[16]:


# Calculate the number of students per school with reading scores of 70 or higher from school_data_complete
students_passing_reading = school_data_complete[school_data_complete["reading_score"] >= 70]
school_students_passing_reading = students_passing_reading.groupby(["school_name"]).count()["student_name"]


# In[17]:


# Use the provided code to calculate the number of students per school that passed both math and reading with scores of 70 or higher
students_passing_math_and_reading = school_data_complete[
    (school_data_complete["reading_score"] >= 70) & (school_data_complete["math_score"] >= 70)
]
school_students_passing_math_and_reading = students_passing_math_and_reading.groupby(["school_name"]).size()


# In[18]:


# Use the provided code to calculate the passing rates
per_school_passing_math = school_students_passing_math / per_school_counts * 100
per_school_passing_reading = school_students_passing_reading / per_school_counts * 100
overall_passing_rate = school_students_passing_math_and_reading / per_school_counts * 100


# In[19]:


# Create a DataFrame called `per_school_summary` with columns for the calculations above.
per_school_summary = pd.DataFrame({
    "School Type": school_types,
    "Total Students": per_school_counts,
    "Total School Budget": per_school_budget,  
    "Per Student Budget": per_school_capita,
    "Average Math Score": per_school_math,
    "Average Reading Score": per_school_reading,
    "Passing Math Percentage": per_school_passing_math,
    "Passing Reading Percentage": per_school_passing_reading,
    "Overall Passing Rate": overall_passing_rate
})

# Formatting
per_school_summary["Total School Budget"] = per_school_summary["Total School Budget"].map("${:,.2f}".format)
per_school_summary["Per Student Budget"] = per_school_summary["Per Student Budget"].map("${:,.2f}".format)

# Display the DataFrame
per_school_summary


# ## Highest-Performing Schools (by % Overall Passing)

# In[20]:


# Sort the schools by `% Overall Passing` in descending order and display the top 5 rows.
top_schools = per_school_summary.sort_values(["Overall Passing Rate"], ascending=False)
top_schools.head(5)


# ## Bottom Performing Schools (By % Overall Passing)

# In[21]:


# Sort the schools by `% Overall Passing` in descending order and display the top 5 rows.
bottom_schools = per_school_summary.sort_values(["Overall Passing Rate"], ascending=True)
bottom_schools.head(5)


# ## Math Scores by Grade

# In[48]:


# Use the code provided to separate the data by grade
ninth_graders = school_data_complete[(school_data_complete["grade"] == "9th")]
tenth_graders = school_data_complete[(school_data_complete["grade"] == "10th")]
eleventh_graders = school_data_complete[(school_data_complete["grade"] == "11th")]
twelfth_graders = school_data_complete[(school_data_complete["grade"] == "12th")]

# Group by `school_name` and take the mean of the `math_score` column for each.
ninth_grader_math_scores = ninth_graders.groupby(["school_name"]).mean(numeric_only=True)["math_score"]
tenth_grader_math_scores = tenth_graders.groupby(["school_name"]).mean(numeric_only=True)["math_score"]
eleventh_grader_math_scores = eleventh_graders.groupby(["school_name"]).mean(numeric_only=True)["math_score"]
twelfth_grader_math_scores = twelfth_graders.groupby(["school_name"]).mean(numeric_only=True)["math_score"]



# Combine each of the scores above into single DataFrame called `math_scores_by_grade`
math_scores_by_grade = pd.DataFrame({
    "9th": ninth_grader_math_scores,
    "10th": tenth_grader_math_scores,
    "11th": eleventh_grader_math_scores,
    "12th": twelfth_grader_math_scores
})

# Minor data wrangling
math_scores_by_grade = math_scores_by_grade[["9th", "10th", "11th", "12th"]]
math_scores_by_grade.index.name = None

# Display the DataFrame
math_scores_by_grade


# ## Reading Score by Grade 

# In[50]:


# Use the code provided to separate the data by grade
ninth_graders = school_data_complete[(school_data_complete["grade"] == "9th")]
tenth_graders = school_data_complete[(school_data_complete["grade"] == "10th")]
eleventh_graders = school_data_complete[(school_data_complete["grade"] == "11th")]
twelfth_graders = school_data_complete[(school_data_complete["grade"] == "12th")]

# Group by `school_name` and take the mean of the the `reading_score` column for each.
ninth_grader_reading_scores = ninth_graders.groupby(["school_name"]).mean(numeric_only=True)["reading_score"]
tenth_grader_reading_scores = tenth_graders.groupby(["school_name"]).mean(numeric_only=True)["reading_score"]
eleventh_grader_reading_scores = eleventh_graders.groupby(["school_name"]).mean(numeric_only=True)["reading_score"]
twelfth_grader_reading_scores = twelfth_graders.groupby(["school_name"]).mean(numeric_only=True)["reading_score"]

# Combine each of the scores above into single DataFrame called `reading_scores_by_grade`
reading_scores_by_grade = pd.DataFrame({
    "9th": ninth_grader_reading_scores,
    "10th": tenth_grader_reading_scores,
    "11th": eleventh_grader_reading_scores,
    "12th": twelfth_grader_reading_scores
})

# Minor data wrangling
reading_scores_by_grade = reading_scores_by_grade[["9th", "10th", "11th", "12th"]]
reading_scores_by_grade.index.name = None

# Display the DataFrame
reading_scores_by_grade


# ## Scores by School Spending

# In[51]:


# Establish the bins
spending_bins = [0, 585, 630, 645, 680]
labels = ["<$585", "$585-630", "$630-645", "$645-680"]


# In[52]:


# Create a copy of the school summary since it has the "Per Student Budget"
school_spending_df = per_school_summary.copy()


# In[54]:


# Use `pd.cut` to categorize spending based on the bins.
school_spending_df["Spending Ranges (Per Student)"] = pd.cut(per_school_capita, bins=spending_bins, labels=labels)
school_spending_df


# In[57]:


#  Calculate averages for the desired columns.
spending_math_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Average Math Score"].mean(numeric_only=True)
spending_reading_scores = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Average Reading Score"].mean(numeric_only=True)
spending_passing_math = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Passing Math Percentage"].mean(numeric_only=True)
spending_passing_reading = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Passing Reading Percentage"].mean(numeric_only=True)
overall_passing_spending = school_spending_df.groupby(["Spending Ranges (Per Student)"])["Overall Passing Rate"].mean(numeric_only=True)


# In[59]:


# Assemble into DataFrame
spending_summary = pd.DataFrame({
    "Average Math Score": spending_math_scores,
    "Average Reading Score": spending_reading_scores,
    "Average Percentage Passing Math": spending_passing_math,
    "Average Percentage Passing Reading": spending_passing_reading,
    "Average Overall Passing Rate": overall_passing_spending
})

# Display results
spending_summary


# ## Scores by School Size

# In[62]:


# Establish the bins.
size_bins = [0, 1000, 2000, 5000]
labels = ["Small (<1000)", "Medium (1000-2000)", "Large (2000-5000)"]


# In[63]:


# Categorize the spending based on the bins
# Use `pd.cut` on the "Total Students" column of the `per_school_summary` DataFrame.

per_school_summary["School Size"] = pd.cut(per_school_counts, bins=size_bins, labels=labels)
per_school_summary


# In[67]:


# Calculate averages for the desired columns.
size_math_scores = per_school_summary.groupby(["School Size"])["Average Math Score"].mean(numeric_only=True)
size_reading_scores = per_school_summary.groupby(["School Size"])["Average Reading Score"].mean(numeric_only=True)
size_passing_math = per_school_summary.groupby(["School Size"])["Passing Math Percentage"].mean(numeric_only=True)
size_passing_reading = per_school_summary.groupby(["School Size"])["Passing Reading Percentage"].mean(numeric_only=True)
size_overall_passing = per_school_summary.groupby(["School Size"])["Overall Passing Rate"].mean(numeric_only=True)


# In[68]:


# Create a DataFrame called `size_summary` that breaks down school performance based on school size (small, medium, or large).
# Use the scores above to create a new DataFrame called `size_summary`
size_summary = pd.DataFrame({
    "Average Math Score": size_math_scores,
    "Average Reading Score": size_reading_scores,
    "Passing Math Percentage": size_passing_math,
    "Passing Reading Percentage": size_passing_reading,
    "Overall Passing Rate": size_overall_passing
})

# Display results
size_summary


# ## Scores by School Type

# In[71]:


# Group the per_school_summary DataFrame by "School Type" and average the results.
average_math_score_by_type = per_school_summary.groupby(["School Type"])["Average Math Score"].mean(numeric_only=True)
average_reading_score_by_type = per_school_summary.groupby(["School Type"])["Average Reading Score"].mean(numeric_only=True)
average_percent_passing_math_by_type = per_school_summary.groupby(["School Type"])["Passing Math Percentage"].mean(numeric_only=True)
average_percent_passing_reading_by_type = per_school_summary.groupby(["School Type"])["Passing Reading Percentage"].mean(numeric_only=True)
average_percent_overall_passing_by_type = per_school_summary.groupby(["School Type"])["Overall Passing Rate"].mean(numeric_only=True)


# In[72]:


# Assemble the new data by type into a DataFrame called `type_summary`
type_summary = pd.DataFrame({
    "Average Math Score": average_math_score_by_type,
    "Average Reading Score": average_reading_score_by_type,
    "Passing Math Percentage": average_percent_passing_math_by_type,
    "Passing Reading Percentage": average_percent_passing_reading_by_type,
    "Overall Passing Rate": average_percent_overall_passing_by_type
})

# Display results
type_summary