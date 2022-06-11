# Jeopardy-Data

#### Grace Olson, Jack Lloyd (June, 2022)

## **Proposal:**
The motivation for this analysis was to demonstrate the ETL process: extracting data from various sources, transforming the data by cleaning and restructuring it to make it consumable by the end user, and loading the transformed data to an appropriate database. 

Jeopardy! is one of America's favorite quiz game shows and a household classic with lots of data on past shows circulating the internet. Thus, for this ETL analysis we used Kaggle/Github and Reddit to find Jeopardy! datasets on questions asked, the results of each show, and the contestants.
* From Reddit, we used the json data file containing 200,000+ Jeopardy! questions: https://www.reddit.com/r/datasets/comments/1uyd0t/200000_jeopardy_questions_in_a_json_file/
* From Kaggle/Github, we used three csv files containing Jeopardy! questions asked, the outcome of Final Jeopardy!, and the contestants: https://github.com/CNuge/kaggle-code/tree/master/jeopardy
    * We have two datasets containing the questions asked because the second data set (the csv) contains interesting data on where the question was located on the board, while the first questions file (the json) does not. 

The information found on these websites can be used to analyze Jeopardy! data and answer questions for some of the show's biggest fans, such as:
1. What are the most common question categories on Jeopardy!?
2. Is there a strategy for a question to select first based on its location on the board and previous successes?
3. What is the average wager in Final Jeopardy!
4. Is the average wager higher among returning champions than for the two other players?
5. Are a player's positon and the final result of the show related?
6. What position gets Final Jeopardy! correct most often? Is there a relationship between the two?
7. What city/state produces the most Jeopardy! contestants? The least?
8. What city/state produces the most Jeopardy! winners? The least?
9. What is the most a contestant wagered in Final Jeopardy! and won? And lost?
10. What occupation is most common among Jeopardy! contestants?

## **ETL: Extract**
Extracting the data includes downloading the datasets from Reddit and Kaggle/Github, saving them to a Resources folder, setting up a jupyter notebook, and reading in the files from their current format (json or csv) to a Pandas Dataframe
* Step 1: Create a 'Jeopardy_Data' folder in your desired directory. Add a 'Resources' sub-folder within this folder. 
* Step 2: Download the 'JEOPARDY_QUESTIONS1.json' file from the Reddit link above and the contestants.csv, final_results.csv, and questions.csv from the GitHub link above and save to the 'Resources' folder.
* Step 3: Create a juptyter notebook from which to work in for the remainder of the analysis. Import the following dependencies:
    * import pandas as pd
    * import json
    * from sqlalchemy import create_engine
    * from sqlalchemy_utils import database_exists, create_database
    * from config import password
        * You will need to create a config.py file in your 'Jeopardy_Data' folder with a variable called 'password'. Set this variable equal to your PGadmin password. This will enable you to create a connection to postgres database in PGAdmin and load your cleaned datasets to the database. 
* Step 4: Use pandas to read in the json and csv files respectively and create a DataFrame. 
    * The first questions DataFrame should have 216,930 rows and 7 columns
    * The second questions DataFrame should have 225,769 rows and 9 columns
    * The final_results DataFrame should have 12,012 rows and 7 columns
    * The contestants DataFrame should have 12,012 rows and 6 columns

## **ETL: Transform**
Transforming the data involves cleaning the DataFrames to provide the end user with data that is able to be read, used, analyzed, merged, joined, etc. to answer their questions
* Transform the first 'questions' DataFrame:
    * Rename the 'show_number' column to 'game_id' to match other DataFrames
    * Reorder columns so game_id is first and column order makes sense (air_date before round, round before category, category before question)
    * Change air_date data type to datetime
    * Replace the null values in the 'value' column with $0. This makes sense because these rows are for the Final Jeopardy! rounds where contestants determine the value with their wager. 
    * This DataFrame is now ready to be used for analysis!
 
* Transform the second 'questions' DataFrame:
    * Rename 'question_text' column to 'question' to match other DataFrames
    * Replce the -1 in the 'value' column to 0 to match other DataFrames. Again, this makes sense since these rows are for the Final Jeopardy! rounds where contestants determine the value with their wager. 
    * Replace 'final' in the 'round' column with 'F' to match the format of the rest of the column
    * Replace 'final' in the category column with 'FINAL JEOPARDY' to match the format of the rest of the column
    * Replace the null values in the 'row' and 'column' columns with 0. This makes sense because these rows are for the Final Jeopardy! rounds where the question is not displayed on the game board. 
    * Change the data type of the 'row' and 'column' columns from a float to an integer to remove unnecessary decimals
    * Drop the two rows with null values in the answer column
    * This DataFrame is now ready to be used for analysis!

* Transform the final_results DataFrame:
    * Remove 'dj_score' and 'coryat_score' from the DataFrame since we do not have documentation on what these columns mean
    * Drop any rows with null values in the 'wager' and 'correct' columns. These rows likely indicate a contestant did not make it to the Final Jeopardy! round, thus for our purposes and the questions we are interested in, we do not need these rows. 
    * Change the data type of the 'wager' column from a float to an integer to remove unnecessary decimals 
    * This DataFrame is now ready to be used for analysis!

* Transform the contestants DataFrame: 
    * Drop any rows with null values in any column- many contestants did not report a hometown city/state or occupation, thus for our purposes and the questions we are interested in, we do not need these rows. 
    * This DataFrame is now ready to be used for analysis!

## **ETL: Load**
The last step in the ETL process is loading the cleaned DataFrames to a database to be used by the end user for future analysis or business application. For these datasets, we suggest a relational database such as postgres because we have cleaned the data and it is now in tabular format. Simply create an engine to connect to the database and use pandas to send the DataFrames to the database. 
* Step 1: Create a variable, 'engine', that connects to your postgres database. Here is where the config.py file we created earlier with your PGadmin password comes into play. 
* Step 2: Use an if statement to create the database if it does not already exist. 
* Step 3: Use the pandas method .to_sql to create tables from our pandas DataFrames in postgres. 
* Step 4: View the tables in our pur Jeopardy_db by opening PGAdmin and enting your password. 

## **Takeaways/Further Analysis:**
These four DataFrames are now loaded to our postgres database and ready to use for analysis. We would recommend joining the two questions tables by joining on either question or game_id. However, we kept them separate here to give the end user the liberty to join based on their desired questions. 

ETL is a great framework for working with data and preparing it for analysis or use by an end user or business--it helps to create a mind map of the first three steps required for analysis. In this project, we  successfully extracted Jeopardy! data from two data sources on the web, cleaned the data sources using Pandas, and loaded the cleaned DataFrames to a relational database (postgres) for others to leverage. 