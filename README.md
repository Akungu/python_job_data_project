<<<<<<< HEAD
# Overview
The purpose of the study is to analyse job data, specifically for data analysis roles in the United States, so as to gain insight into the most paying jobs and skills, most demanded skills, and the most optimal jobs (high slary meets high demand skills).

# Background
The source of the dataset is [ Python for Data Analytics ]('lukebarousse/data_jobs'). It contains insights such as job title, salaries, locations and skills.

### The questions I wanted to ask through my SQL queries were :
1. What are the median salary for the top job titles in the United States ?

2. What are the most in-demand skills for data nalysts, data engineers and data scientists ?

3. What are the most optimal skills to learn for data analysts?

4. What are the top-paid and most in-demand skills for data analysts ?

5. What are the skills trend for data analysts?

# Tools I used
- **Python**: The backbone of my analysis, allowing me to analyze the data and find critical insights.I also used the following Python libraries:
  - **Pandas Library**: This was used to analyze the data.
  - **Matplotlib Library**: I used it to visualize the data.
  - **Seaborn Library**: Helped me create more advanced visuals.
  - **Jupyter Notebooks**: The tool I used to run my Python scripts which let me easily include my notes and analysis.
  - **Visual Studio Code**: My go-to for executing my Python scripts.
  - **Git & GitHub**: Essential for version control and sharing my Python code and analysis, ensuring collaboration and project tracking.

# Data Cleaning
## Import and Clean the Data
```python
# import libraries
import pandas as pd
import matplotlib.pyplot as plt
import ast
from datasets import load_dataset
import seaborn as sns

# load dataset
dataset = load_dataset('lukebarousse/data_jobs')
df = dataset['train'].to_pandas()


# clean dataset
df['job_posted_date'] = pd.to_datetime(df['job_posted_date'])
df['job_skills'] = df['job_skills'].apply(lambda x: ast.literal_eval(x) if pd.notna(x) else x)
```
## Filter the Data
This is so that I focus only on job postings in the United States.

```python
df_us = df[df['job_country'] == 'United States'].copy()
```
# Analysis
### 1. What are the median salary for the top job titles in the United States ?
I looked at and grouped the data by the  the top six job_title_shorts, and found the median salary for each job_title_short. I then used seaborn to generate the box plots for each.

```python
job_titles = df_us['job_title_short'].value_counts().index[:6].to_list()
job_titles

df_us_top_six = df_us[df_us['job_title_short'].isin(job_titles)]

sorted_jobs = df_us_top_six.groupby('job_title_short')['salary_year_avg'].median().sort_values(ascending=False).index

sns.boxplot(data=df_us_top_six, vert=False, x='salary_year_avg', y='job_title_short', order=sorted_jobs)

plt.title('Salary Distribution in the United States')
plt.ylabel('')
plt.xlim(0, 600000)
```
### Results
![median_salary](assets/median_salary.png)
### Insights
- Data scientists and engineers earn more as compared to data analysts. It is therefore advisable for one to up- skill and specialize more  in case they want to earn more.

### 2. What are the most in-demand skills for data anlysts, data engineers and data scientists ?

I exploded the skills because they are in lists. I then grouped by the skills and found their size and sorted them in descending order.

```python
df_skills = df_us.explode('job_skills')
df_skills_grouped = df_skills.groupby(['job_skills','job_title_short']).size()
df_total_skills = df_skills_grouped.reset_index(name='total_skills_count')
df_final_skills = df_total_skills.sort_values(by='total_skills_count', ascending=False)
df_percentage = df_us['job_title_short'].value_counts().reset_index(name='total_job_counts')
df_skills_percentage = pd.merge(df_final_skills, df_percentage, how='left', on='job_title_short')
df_skills_percentage['skill_percent'] = 100 * df_skills_percentage['total_skills_count']/ df_skills_percentage['total_job_counts']

job_titles = ['Data Analyst', 'Data Engineer', 'Data Scientist']

fig, ax = plt.subplots(3,1)

for i, job_title in enumerate (job_titles):
   df_pivot =  df_skills_percentage[df_skills_percentage['job_title_short'] == job_title].head(5)
   #df_pivot.plot(kind='barh', x='job_skills', y='skill_percent', ax=ax[i], title = job_title)
   sns.barplot(data=df_pivot, x='skill_percent', y='job_skills', ax=ax[i], hue='skill_percent', palette='dark:r_r')
   ax[i].set_title(job_title)
   ax[i].set_ylabel('') 
   ax[i].set_xlabel('')
   ax[i].legend().set_visible(False)
   ax[i].set_xlim(0, 75)
fig.suptitle('Percentage of top skills in job postings in the United States', fontsize=17)
fig.tight_layout(h_pad=0.5)
plt.show()
```
### Results
![top_skills](assets/top_skills.png)

### Inights
- SQL, Python, Excel and Tableau are must have skills for data analysts with SQL being the most in demand.
- SQL and Python are also very much in high-demand for data science and data engineer roles. 
- It is also important to learn cloud technology skills such as azure, aws and spark as they come in handy for data engineer roles.

### 3. What are the most optimal skills to learn for data analysts?

These are the most demanded and top paying skills as well. I used median salary to get the most paying skills.

```python
df_exploded =df_data_analysis_skills.explode('job_skills')

df_skills = df_exploded.groupby('job_skills').agg(
    skill_count = ('job_skills', 'count'),
    median_salary = ('salary_year_avg', 'median')
)

df_total_skill_count =df_skills.sort_values(by= 'skill_count', ascending=False).head(10)

df_total_skill_count.plot(kind='scatter', x='skill_count', y='median_salary')

for i,txt in enumerate(df_total_skill_count.index):
    plt.text(df_total_skill_count['skill_count'].iloc[i], df_total_skill_count['median_salary'].iloc[i], txt)

plt.xlabel('Count of Job Postings')
plt.ylabel('Median Yearly Salary')
plt.title('Salary vs Count of Job Postings for Top 10 Skills')
```
### Results
![optimal_skills](assets/optimal.png)

### Insights
- Python is the most optimal skill for a data analyst to learn.
- Other top conteders are tableau, R and SQL.
- One should also be familiar with BI tools ( power bi ), excel, and word. 

### 4. What are the top-paid and most in-demand skills for data analysts ?

```python
fig, ax = plt.subplots(2, 1)

# Highest paid
sns.barplot(data=df_top_paying, x='median', y=df_top_paying.index,
            ax=ax[0], palette='dark:r')
ax[0].set_title('Top 10 Highest Paid Skills for Data Analysts')
ax[0].set_ylabel('')
ax[0].set_xlabel('Median Salary')

# Most in-demand
df_popular_skills_sorted = df_popular_skills.sort_values(by='count', ascending= False)
sns.barplot(data=df_popular_skills_sorted, x='count', y=df_popular_skills_sorted.index,
            ax=ax[1], palette='dark:r')
ax[1].set_title('Top 10 Most In-demand Skills for Data Analysts')
ax[1].set_ylabel('')
ax[1].set_xlabel('Job_Counts')

fig.tight_layout()
```
### Results
![skill_pay](assets/skill_pay.png)

### Insights
- Specialization skills which are not most-demanded for are the top-most-paying ones. It is thefefore safe to say that learning one of these skills could be very benefical.
- SQL, Excel, Python, Tableau are still the most in demand skills for data analysts although they are not the top paying.

### 5. What are the skills trend for data analysts?

I grouped the skills for 2023 by month so as to notice a pattern.

```python
df_data_analysis_explode_pivot = df_data_analysis_explode.pivot_table(
    index='month_number',
    columns='job_skills',
    aggfunc='size',
    fill_value=0
)

df_data_analysis_explode_pivot.loc['total'] = df_data_analysis_explode_pivot.sum()
df_data_analysis_explode_pivot = df_data_analysis_explode_pivot[df_data_analysis_explode_pivot.loc['total'].sort_values(ascending=False).index]

df_data_analysis_explode_pivot = df_data_analysis_explode_pivot.drop('total')
df_data_analysis_explode_pivot

df_data_analysis_explode_pivot.iloc[:,:5].plot(kind='line')
```
### Results
![trend](assets/trend.png)

### Insights
- The peak for the skills - sql, excel, python, tablaeu and power bi - are in the motnhs of January and August. These implies that it is in this 2 months that most companies look for personnel with these skills.
- SQL was the most demanded (trending) skill for data analysts in 2023.

# Lessons Learnt
I learnt how to leverage python for data analysis. Using differnt libraries such as matplotlib, seaborn, and pandas to manipulate raw data.
I also learnt the skills I required to venture into the data analysis job market and the salary distributions for differnent roles.






=======
Overview
Welcome to my analysis of the data job market, focusing on data analyst roles. This project was created out of a desire to navigate and understand the job market more effectively. It delves into the top-paying and in-demand skills to help find optimal job opportunities for data analysts.

The data sourced from Luke Barousse's Python Course which provides a foundation for my analysis, containing detailed information on job titles, salaries, locations, and essential skills. Through a series of Python scripts, I explore key questions such as the most demanded skills, salary trends, and the intersection of demand and salary in data analytics.

The Questions
Below are the questions I want to answer in my project:

What are the skills most in demand for the top 3 most popular data roles?
How are in-demand skills trending for Data Analysts?
How well do jobs and skills pay for Data Analysts?
What are the optimal skills for data analysts to learn? (High Demand AND High Paying)
Tools I Used
For my deep dive into the data analyst job market, I harnessed the power of several key tools:

Python: The backbone of my analysis, allowing me to analyze the data and find critical insights.I also used the following Python libraries:
Pandas Library: This was used to analyze the data.
Matplotlib Library: I visualized the data.
Seaborn Library: Helped me create more advanced visuals.
Jupyter Notebooks: The tool I used to run my Python scripts which let me easily include my notes and analysis.
Visual Studio Code: My go-to for executing my Python scripts.
Git & GitHub: Essential for version control and sharing my Python code and analysis, ensuring collaboration and project tracking.
Data Preparation and Cleanup
This section outlines the steps taken to prepare the data for analysis, ensuring accuracy and usability.

Import & Clean Up Data
I start by importing necessary libraries and loading the dataset, followed by initial data cleaning tasks to ensure data quality.

# Importing Libraries
import ast
import pandas as pd
import seaborn as sns
from datasets import load_dataset
import matplotlib.pyplot as plt  

# Loading Data
dataset = load_dataset('lukebarousse/data_jobs')
df = dataset['train'].to_pandas()

# Data Cleanup
df['job_posted_date'] = pd.to_datetime(df['job_posted_date'])
df['job_skills'] = df['job_skills'].apply(lambda x: ast.literal_eval(x) if pd.notna(x) else x)
Filter US Jobs
To focus my analysis on the U.S. job market, I apply filters to the dataset, narrowing down to roles based in the United States.

df_US = df[df['job_country'] == 'United States']
The Analysis
Each Jupyter notebook for this project aimed at investigating specific aspects of the data job market. Here’s how I approached each question:

1. What are the most demanded skills for the top 3 most popular data roles?
To find the most demanded skills for the top 3 most popular data roles. I filtered out those positions by which ones were the most popular, and got the top 5 skills for these top 3 roles. This query highlights the most popular job titles and their top skills, showing which skills I should pay attention to depending on the role I'm targeting.

View my notebook with detailed steps here: 2_Skill_Demand.

Visualize Data
fig, ax = plt.subplots(len(job_titles), 1)


for i, job_title in enumerate(job_titles):
    df_plot = df_skills_perc[df_skills_perc['job_title_short'] == job_title].head(5)[::-1]
    sns.barplot(data=df_plot, x='skill_percent', y='job_skills', ax=ax[i], hue='skill_count', palette='dark:b_r')

plt.show()
Results
Likelihood of Skills Requested in the US Job Postings

Bar graph visualizing the salary for the top 3 data roles and their top 5 skills associated with each.

Insights:
SQL is the most requested skill for Data Analysts and Data Scientists, with it in over half the job postings for both roles. For Data Engineers, Python is the most sought-after skill, appearing in 68% of job postings.
Data Engineers require more specialized technical skills (AWS, Azure, Spark) compared to Data Analysts and Data Scientists who are expected to be proficient in more general data management and analysis tools (Excel, Tableau).
Python is a versatile skill, highly demanded across all three roles, but most prominently for Data Scientists (72%) and Data Engineers (65%).
2. How are in-demand skills trending for Data Analysts?
To find how skills are trending in 2023 for Data Analysts, I filtered data analyst positions and grouped the skills by the month of the job postings. This got me the top 5 skills of data analysts by month, showing how popular skills were throughout 2023.

View my notebook with detailed steps here: 3_Skills_Trend.

Visualize Data
from matplotlib.ticker import PercentFormatter

df_plot = df_DA_US_percent.iloc[:, :5]
sns.lineplot(data=df_plot, dashes=False, legend='full', palette='tab10')

plt.gca().yaxis.set_major_formatter(PercentFormatter(decimals=0))

plt.show()
Results
Trending Top Skills for Data Analysts in the US
Bar graph visualizing the trending top skills for data analysts in the US in 2023.

Insights:
SQL remains the most consistently demanded skill throughout the year, although it shows a gradual decrease in demand.
Excel experienced a significant increase in demand starting around September, surpassing both Python and Tableau by the end of the year.
Both Python and Tableau show relatively stable demand throughout the year with some fluctuations but remain essential skills for data analysts. Power BI, while less demanded compared to the others, shows a slight upward trend towards the year's end.
3. How well do jobs and skills pay for Data Analysts?
To identify the highest-paying roles and skills, I only got jobs in the United States and looked at their median salary. But first I looked at the salary distributions of common data jobs like Data Scientist, Data Engineer, and Data Analyst, to get an idea of which jobs are paid the most.

View my notebook with detailed steps here: 4_Salary_Analysis.

Visualize Data
sns.boxplot(data=df_US_top6, x='salary_year_avg', y='job_title_short', order=job_order)

ticks_x = plt.FuncFormatter(lambda y, pos: f'${int(y/1000)}K')
plt.gca().xaxis.set_major_formatter(ticks_x)
plt.show()
Results
Salary Distributions of Data Jobs in the US
Box plot visualizing the salary distributions for the top 6 data job titles.

Insights
There's a significant variation in salary ranges across different job titles. Senior Data Scientist positions tend to have the highest salary potential, with up to $600K, indicating the high value placed on advanced data skills and experience in the industry.

Senior Data Engineer and Senior Data Scientist roles show a considerable number of outliers on the higher end of the salary spectrum, suggesting that exceptional skills or circumstances can lead to high pay in these roles. In contrast, Data Analyst roles demonstrate more consistency in salary, with fewer outliers.

The median salaries increase with the seniority and specialization of the roles. Senior roles (Senior Data Scientist, Senior Data Engineer) not only have higher median salaries but also larger differences in typical salaries, reflecting greater variance in compensation as responsibilities increase.

Highest Paid & Most Demanded Skills for Data Analysts
Next, I narrowed my analysis and focused only on data analyst roles. I looked at the highest-paid skills and the most in-demand skills. I used two bar charts to showcase these.

Visualize Data
fig, ax = plt.subplots(2, 1)  

# Top 10 Highest Paid Skills for Data Analysts
sns.barplot(data=df_DA_top_pay, x='median', y=df_DA_top_pay.index, hue='median', ax=ax[0], palette='dark:b_r')

# Top 10 Most In-Demand Skills for Data Analystsr')
sns.barplot(data=df_DA_skills, x='median', y=df_DA_skills.index, hue='median', ax=ax[1], palette='light:b')

plt.show()
Results
Here's the breakdown of the highest-paid & most in-demand skills for data analysts in the US:

The Highest Paid & Most In-Demand Skills for Data Analysts in the US Two separate bar graphs visualizing the highest paid skills and most in-demand skills for data analysts in the US.

Insights:
The top graph shows specialized technical skills like dplyr, Bitbucket, and Gitlab are associated with higher salaries, some reaching up to $200K, suggesting that advanced technical proficiency can increase earning potential.

The bottom graph highlights that foundational skills like Excel, PowerPoint, and SQL are the most in-demand, even though they may not offer the highest salaries. This demonstrates the importance of these core skills for employability in data analysis roles.

There's a clear distinction between the skills that are highest paid and those that are most in-demand. Data analysts aiming to maximize their career potential should consider developing a diverse skill set that includes both high-paying specialized skills and widely demanded foundational skills.

4. What are the most optimal skills to learn for Data Analysts?
To identify the most optimal skills to learn ( the ones that are the highest paid and highest in demand) I calculated the percent of skill demand and the median salary of these skills. To easily identify which are the most optimal skills to learn.

View my notebook with detailed steps here: 5_Optimal_Skills.

Visualize Data
from adjustText import adjust_text
import matplotlib.pyplot as plt

plt.scatter(df_DA_skills_high_demand['skill_percent'], df_DA_skills_high_demand['median_salary'])
plt.show()
Results
Most Optimal Skills for Data Analysts in the US
A scatter plot visualizing the most optimal skills (high paying & high demand) for data analysts in the US.

Insights:
The skill Oracle appears to have the highest median salary of nearly $97K, despite being less common in job postings. This suggests a high value placed on specialized database skills within the data analyst profession.

More commonly required skills like Excel and SQL have a large presence in job listings but lower median salaries compared to specialized skills like Python and Tableau, which not only have higher salaries but are also moderately prevalent in job listings.

Skills such as Python, Tableau, and SQL Server are towards the higher end of the salary spectrum while also being fairly common in job listings, indicating that proficiency in these tools can lead to good opportunities in data analytics.

Visualizing Different Techonologies
Let's visualize the different technologies as well in the graph. We'll add color labels based on the technology (e.g., {Programming: Python})

Visualize Data
from matplotlib.ticker import PercentFormatter

# Create a scatter plot
scatter = sns.scatterplot(
    data=df_DA_skills_tech_high_demand,
    x='skill_percent',
    y='median_salary',
    hue='technology',  # Color by technology
    palette='bright',  # Use a bright palette for distinct colors
    legend='full'  # Ensure the legend is shown
)
plt.show()
Results
Most Optimal Skills for Data Analysts in the US with Coloring by Technology
A scatter plot visualizing the most optimal skills (high paying & high demand) for data analysts in the US with color labels for technology.

Insights:
The scatter plot shows that most of the programming skills (colored blue) tend to cluster at higher salary levels compared to other categories, indicating that programming expertise might offer greater salary benefits within the data analytics field.

The database skills (colored orange), such as Oracle and SQL Server, are associated with some of the highest salaries among data analyst tools. This indicates a significant demand and valuation for data management and manipulation expertise in the industry.

Analyst tools (colored green), including Tableau and Power BI, are prevalent in job postings and offer competitive salaries, showing that visualization and data analysis software are crucial for current data roles. This category not only has good salaries but is also versatile across different types of data tasks.

What I Learned
Throughout this project, I deepened my understanding of the data analyst job market and enhanced my technical skills in Python, especially in data manipulation and visualization. Here are a few specific things I learned:

Advanced Python Usage: Utilizing libraries such as Pandas for data manipulation, Seaborn and Matplotlib for data visualization, and other libraries helped me perform complex data analysis tasks more efficiently.
Data Cleaning Importance: I learned that thorough data cleaning and preparation are crucial before any analysis can be conducted, ensuring the accuracy of insights derived from the data.
Strategic Skill Analysis: The project emphasized the importance of aligning one's skills with market demand. Understanding the relationship between skill demand, salary, and job availability allows for more strategic career planning in the tech industry.
Insights
This project provided several general insights into the data job market for analysts:

Skill Demand and Salary Correlation: There is a clear correlation between the demand for specific skills and the salaries these skills command. Advanced and specialized skills like Python and Oracle often lead to higher salaries.
Market Trends: There are changing trends in skill demand, highlighting the dynamic nature of the data job market. Keeping up with these trends is essential for career growth in data analytics.
Economic Value of Skills: Understanding which skills are both in-demand and well-compensated can guide data analysts in prioritizing learning to maximize their economic returns.
Challenges I Faced
This project was not without its challenges, but it provided good learning opportunities:

Data Inconsistencies: Handling missing or inconsistent data entries requires careful consideration and thorough data-cleaning techniques to ensure the integrity of the analysis.
Complex Data Visualization: Designing effective visual representations of complex datasets was challenging but critical for conveying insights clearly and compellingly.
Balancing Breadth and Depth: Deciding how deeply to dive into each analysis while maintaining a broad overview of the data landscape required constant balancing to ensure comprehensive coverage without getting lost in details.
Conclusion
This exploration into the data analyst job market has been incredibly informative, highlighting the critical skills and trends that shape this evolving field. The insights I got enhance my understanding and provide actionable guidance for anyone looking to advance their career in data analytics. As the market continues to change, ongoing analysis will be essential to stay ahead in data analytics. This project is a good foundation for future explorations and underscores the importance of continuous learning and adaptation in the data field.
>>>>>>> d3109fd45ee6430386bcb18ee0837387fd8be35b
