# Web Traffic Transformation README
## Project Description
In this project, I created a python script in order to transform web traffic data stored in time-record format, each row is a page view into a per-user format where each row is a different user and the
columns represent the time spent on each of the pages. 

## Installation

This script requires you to import the `pandas` library  
Documentation: https://pandas.pydata.org/docs/user_guide/index.html#user-guide

## Usage

1. Automatically extract the csv's from the public root URL: https://public.wiwdata.com/engineering-challenge/data/ (26 total)
2. Aggregate the data tables into one dataframe and transform data into a pivot table
3. Load the results csv locally

## Code Breakdown

My code includes 3 functions, one to extract the csv from the root URL, another to format the dataset, and lastly to load the results.

Since we want to be able to changed the rool URL later, I defined the base URL separately:  
`base_url = 'https://public.wiwdata.com/engineering-challenge/data/'`  

### Extracting the data:

This function below takes in the root URL as argument and creates an empty dataframe in order to combine all future datasets from the for loop.
While looping through the ascii alphabets, it creates the complete file path by combining the e.g. `'a.csv'` file to the end of the `base_url`.  
The try-except statements provides us the opportunities to identify any datasets that weren't able to be aggregated. 


```
def extract_files_from_s3(url):
  # make an empty data frame
  table = pd.DataFrame([])
  # ascii_lovercase function returns all alphabets characters in lower case
  for character in ascii_lowercase:
    try:
      complete_url = f'{url}{character}.csv'
      # pandas to read the csv file from the completed url
      df = pd.read_csv(complete_url)
      # concat the new data with the previuws data
      table = pd.concat([table, df])
    except:
      # if there is an error in reading the file then log the error
      print(f'Error in reading the {character}.csv file')
  return table`  
 ```

### Transforming the data:
This function simply transforms the aggregated dataframe frame into a pivot table format. The pivot table contains one row for each `user_id` and has columns populated with the `length` of time each user spent on each `path`. If there were no time spent on a particular `path`, then the default value is 0. 

```
def format_output_for_csv(table):
  if not table.empty:
    # pandas make pivot table bases upon the sum of the path of again userid
    pivot_table = pd.pivot_table(table, 
                           index=["user_id"], 
                           columns=["path"], 
                           values="length", 
                           aggfunc="sum", 
                           fill_value=0)
    return pivot_table
 ```
 
 ### Loading the data:

The program writes out a standard csv file that can be opened in Excel for review.
 
```
def load_csv_locally(pivot_table):
  # export file in result .csv
  pivot_table.to_csv(path_or_buf='result.csv')
  print("csv exported successfully")

```

### Main function:

The main function calls all of the other functions sequentially. The reason why a new variable is defined for each function call is because this method provides us the ability to debug any error should they arise. As opposed to `load_csv_locally(format_output_for_csv(extract_files_from_s3(base_url)))`.


```
def main():
  # call export csv method
  combined_table = extract_files_from_s3(base_url)
  pivot_table = format_output_for_csv(combined_table)
  load_csv_locally(pivot_table)
```


## Success Criteria
1. The program should be designed so that the root URL could be changed later and the
program re-run on new data. That means downloading the data must be done within the
program.
2. The program should write out to a standard CSV file that can be opened in Excel for
review.
3. The program should be written in python3.
4. The program should be available in a public GitHub or GitLab account with
documentation on how to install and run it.



