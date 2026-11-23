# Data Professional Survey — Career Backgrounds and Job Satisfaction

This Power BI project explores data careers from two perspectives: the backgrounds and entry experiences of survey respondents, and their satisfaction after entering the field. I created two dashboard pages to examine these perspectives separately.

The data preparation uses Power Query to remove empty fields, consolidate categorical responses, convert salary ranges into numeric estimates, and reshape satisfaction scores into a long-format table.

## Download and Explore the Report

[Download the Power BI report (.pbix)](Data_Professional_Survey.pbix?raw=true)

### Open the Report

1. Download `Data_Professional_Survey.pbix` using the link above.
2. Open the file in Microsoft Power BI Desktop on Windows.
3. Switch between the two report pages to explore career backgrounds and job satisfaction.
4. Use the available slicers and select chart elements to explore the report’s filtering interactions.

The report contains imported data, so you can explore the saved report without refreshing it or connecting to Power BI Service. A Power BI Service account is not required to view the saved report in Power BI Desktop.

To refresh the data, download the original Excel workbook listed under **References and Acknowledgments**, then update its file path in **Transform data > Data source settings** before refreshing.

## Data Import

The `Data Professional Survey` worksheet in `Power BI - Final Project.xlsx` contains 630 survey responses, covering job titles, salary ranges, industries, programming language preferences, job satisfaction, age, and country of residence.

After importing the worksheet, I promoted the first row to headers and assigned appropriate data types. Identifiers and categorical responses were kept as text, age and satisfaction scores were set to whole numbers, and date and time fields were assigned their corresponding types.

## Data Cleaning in Power Query

### 1. Remove Empty Columns

**Action:** Used **Remove Columns** to delete `Browser`, `OS`, `City`, `Country`, and `Referrer`.

**Reason:** All five columns were empty across the 630 responses. They provided no information for grouping, comparison, or calculation, so removing them reduced unnecessary fields and made the dataset easier to work with.

The removed `Country` field was an empty system field. The separate survey question, `Q11 - Which Country do you live in?`, contained actual responses and was retained.

### 2. Simplify Job Title Categories

**Action:** Split `Q1 - Which Title Best Fits your Current Role?` using the opening parenthesis `(` as the delimiter. Kept the first resulting column and removed the column containing supplementary text.

For example, `Other (Please Specify):Account manager` was grouped under the `Other` category.

**Reason:** Respondents who selected “Other” could enter their own job titles, creating many distinct text values within the same survey option. Keeping the main category prevented these descriptions from becoming separate groups and made comparisons across job categories easier.

This transformation consolidated free-text job titles under “Other”; it did not individually reclassify those titles.

### 3. Simplify Programming Language Responses

**Action:** Split `Q5 - Favorite Programming Language` using a colon `:` as the delimiter. Retained the main option before the colon and removed the free-text portion after it.

For example, `Other:Altery` became `Other`.

**Reason:** Free-text answers can fragment a category into many small groups. Retaining the main survey options provided a consistent basis for summarizing programming language preferences.

### 4. Convert Salary Ranges into Numeric Estimates

**Action:**

1. Duplicated `Q3 - Current Yearly Salary (in USD)` to preserve the original salary ranges for reference.
2. Used **Split Column > By Digit to Non-Digit** on the duplicated column to separate the lower bound and the text containing the upper bound.
3. Used **Replace Values** to remove `k` and `-` from the upper-bound text, and removed the unnecessary text column produced by the split.
4. For the open-ended `225k+` category, replaced the extracted `+` with `225` as the assumed upper bound for this calculation.
5. Converted both bounds to whole numbers and created a custom column, `AVG Salary`, using the midpoint formula below.
6. Removed the temporary bound columns and placed `AVG Salary` next to the original salary-range column for easier comparison.

```text
AVG Salary = (Lower Bound + Upper Bound) / 2
```

For example, `41k-65k` was converted to:

```text
(41 + 65) / 2 = 53 thousand USD per year
```

**Reason:** Salary ranges were stored as text and could not be directly used in numeric calculations. A range midpoint provided a numeric estimate for each response, making salary comparisons across groups possible.

**Assumptions and units:** `AVG Salary` is expressed in thousands of USD per year, so `53` represents approximately $53,000 annually. It is an estimate derived from a salary range, rather than a respondent’s exact salary. The `225k+` category was assigned a value of `225`, so the resulting estimate does not capture salary differences above $225,000.

### 5. Simplify Industry Categories

**Action:** Split `Q4 - What Industry do you work in?` using the opening parenthesis `(` as the delimiter. Retained the main industry category and removed the remaining supplementary-text columns.

**Reason:** Free-text descriptions attached to the “Other” option created fragmented categories. Keeping the main options allowed industry responses to be summarized using a consistent set of groups.

### 6. Simplify Job Search Priorities

**Action:** Split `Q8 - If you were to look for a new job today, what would be the most important thing to you?` using the opening parenthesis `(` as the delimiter. Kept the main response category and removed the supplementary-text column.

**Reason:** Consolidating the descriptions attached to “Other” prevented differences in wording from splitting the same survey option into multiple groups. This made the distribution of job search priorities easier to compare.

### 7. Simplify Country of Residence Responses

**Action:** Split `Q11 - Which Country do you live in?` using the opening parenthesis `(` as the delimiter. Kept the main country option and removed the free-text portion.

For example, `Other (Please Specify): Somalia` was grouped under `Other`.

**Reason:** Retaining the main survey options and consolidating “Other” responses made country-level grouping easier.

This transformation removed the specific country names entered under “Other.” That category therefore represents a combined group of countries and should be interpreted accordingly.

### 8. Create an Entry Difficulty Sort Column

**Action:** Duplicated `Q7 - How difficult was it for you to break into Data?`, replaced the response labels with ordered codes, and renamed the new column `difficult  order`.

| Original Response | Sort Code |
| --- | --- |
| Very Easy | 1 |
| Easy | 2 |
| Neither easy nor difficult | 3 |
| Difficult | 4 |
| Very Difficult | 5 |

When replacing text, I processed `Very Easy` before `Easy`, and `Very Difficult` before `Difficult`, to prevent partial replacements from changing the longer labels incorrectly.

**Reason:** The responses have a natural order from easiest to most difficult. Alphabetical sorting would not preserve that order. A separate sort column retained the original labels while providing codes for ordering them consistently.

### 9. Reshape Satisfaction Scores into a Long-Format Table

**Action:**

1. Created a `Satisfaction` query referencing the cleaned `Data Professional Survey` query.
2. Removed fields that were unnecessary for this table, retaining respondent ID, job title, gender, and the three satisfaction measures: salary, work-life balance, and career advancement.
3. Shortened the satisfaction column names to identify each aspect more clearly.
4. Kept `Unique ID`, job title, and gender as identifier columns, then used **Unpivot Other Columns** on the satisfaction measures.
5. Renamed the resulting attribute column to `Satisfaction Aspect` and the value column to `score`.

Before the transformation, each respondent’s satisfaction scores were stored in separate columns. After unpivoting, each row represented one respondent’s score for one satisfaction aspect.

The following example illustrates the structure; it does not contain actual respondent records. Aspect labels are shown in a simplified form.

| Unique ID | Job Title | Gender | Satisfaction Aspect | score |
| --- | --- | --- | --- | --- |
| A001 | Data Analyst | Female | Salary | 6 |
| A001 | Data Analyst | Female | Work-Life Balance | 8 |
| A001 | Data Analyst | Female | Career Advancement | 5 |

**Reason:** A long-format table stores the satisfaction aspect and score in a consistent pair of columns. This makes it easier to aggregate scores across aspects and compare them by job title or gender.

A respondent can appear on multiple rows because each row represents a different satisfaction aspect. These rows are not duplicate survey submissions. Respondent totals should therefore use a distinct count of `Unique ID`.

## Dashboard Analysis

### 1. Career Backgrounds and Entry into the Data Field

**Question:** What do respondents’ roles, education, location, and reported entry difficulty reveal about their career backgrounds?

This dashboard examines job titles, educational qualifications, country of residence, estimated annual salaries, and respondents’ experiences of entering the data field.

**Key findings:**

- **Data Analysts accounted for 381 of 630 respondents (60.5%).** The sample therefore reflects Data Analyst experiences more heavily than those of other roles.
- **Data Scientists had the highest average estimated annual salary, at approximately $93,780.** This group contained 25 respondents, so its average should be interpreted in light of the relatively small sample.
- **The United States had the largest respondent group, with 261 responses (41.4%).** Respondents living in the United States also had the highest average estimated annual salary among the country categories compared, at approximately $78,816.
- **Neutral responses were the largest entry-difficulty group.** Of the 630 respondents, 269 (42.7%) selected “Neither easy nor difficult,” 200 (31.7%) selected “Difficult” or “Very Difficult,” and 161 (25.6%) selected “Easy” or “Very Easy.” These groups were not evenly distributed; difficult responses outnumbered easy responses.

A bachelor’s degree or higher was reported by 526 respondents (83.5% of the full sample); 52 respondents (8.3%) had no recorded education response. However, the educational profile of people who answered this survey does not establish that holding a bachelor’s degree or higher makes entry easier. Assessing that relationship would require comparing reported entry difficulty across education levels; establishing actual entry success would require data on unsuccessful applicants as well.

### 2. Job Satisfaction and Job Search Priorities

**Question:** How satisfied are respondents with their work, and what do they prioritize when considering a new job?

This dashboard focuses on satisfaction with work-life balance, career advancement, and salary, along with country comparisons and job search priorities.

**Satisfaction by aspect:**

| Satisfaction Aspect | Average Score (0–10) |
| --- | ---: |
| Work-Life Balance | 5.74 |
| Career Advancement | 4.76 |
| Salary | 4.27 |

Work-life balance received the highest average score among the three aspects analyzed, followed by career advancement. Salary received the lowest score. Although work-life balance ranked first, its average of 5.74 out of 10 suggests there was still room for improvement.

**Country comparison:**

| Country of Residence | Overall Average Satisfaction Score (0–10) |
| --- | ---: |
| United States | 5.48 |
| United Kingdom | 5.06 |
| India | 4.13 |

For the overall satisfaction measure covering the three selected aspects, the United States ranked highest, followed by the United Kingdom. India had the lowest score among the country categories compared.

**Job search priorities:**

Salary was the most frequently selected priority when considering a new job: 297 respondents (47.1%) selected better salary, compared with 127 (20.2%) selecting remote work. Work-life balance followed closely behind remote work, with 117 responses (18.6%). Salary was selected approximately 2.34 times as often as remote work, although it was not selected by a majority of all respondents. Together with salary’s relatively low satisfaction score, this suggests compensation was an important concern for respondents. Remote work was also valued, but was selected less frequently as the single most important consideration.

## Interpretation Notes

- These findings describe the survey respondents and should not be treated as estimates for the entire data workforce. Data Analysts and respondents living in the United States were especially prominent in the sample.
- Country comparisons use **country of residence**, which does not necessarily identify an employer’s location or the labor market in which a respondent works.
- Salary comparisons use estimates derived from salary-range midpoints. They do not represent exact reported salaries and are not adjusted for cost of living, experience, or differences in job composition.
- Satisfaction rankings in the dashboard refer to the three selected aspects: work-life balance, career advancement, and salary. Across all six satisfaction aspects in the original survey, coworkers received the highest average score (5.86); work-life balance therefore ranked highest only within the three-aspect dashboard comparison.
- The sample includes 90 respondents (14.3%) classified as `Student/Looking/None`; it should not be described as consisting entirely of employed data professionals.
- The country category `Other` contains 224 respondents (35.6%) from multiple countries. It is a combined category rather than a single country.
- Salary and remote-work responses identify respondents’ single most important job search priority; they do not measure how strongly each person values every possible job attribute.

## References and Acknowledgments

The original dataset, [Power BI - Final Project.xlsx](https://github.com/AlexTheAnalyst/Power-BI/blob/main/Power%20BI%20-%20Final%20Project.xlsx), is available in Alex The Analyst’s Power BI repository.
