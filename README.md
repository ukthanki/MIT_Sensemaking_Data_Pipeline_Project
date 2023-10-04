[Go to Back to Home Page](https://ukthanki.github.io/)

# MIT Data Engineering Professional Certification

## Sensemaking Data Pipeline Project

<p align="center">
    <img width="100%" src="https://github.com/ukthanki/MIT_Sensemaking_Data_Pipeline_Project/assets/42117481/80ab1b7e-3d60-455e-978a-70c1ae1662a9">
</p>

Up to this point in the course, we had learned the following topics:
1. NumPy
2. Pandas
3. SQL
4. Linear Regression
5. ETL Fundamentals
6. Flask
7. Java
8. CDC Fundamentals
9. Docker
10. Maven
11. NiFi
12. Hadoop
13. PySpark
14. Airflow
15. Advanced Probability
16. DASK
17. JavaScript

In this project, we used the various skills we have learned to pull unstructured data from the MIT course catalog, clean and analyze the data to determine the word counts throughout all of the course names, and display a visual analysis in a D3 web application. The project was presented in a way that a majority of the base code was already provided to us and we had to fill in various sections with code.

We started in the *assignment.py* file as this was the file that would be used by Airflow. Our fist objective was to define a function called *catalog()* that would be responsible for extracting and storing the unstructured data from a series of URLs into files - one for each course catalog URL. I updated the provided pseudocode to generate the following function:

```python
def catalog():

    def pull(url):
        response = urllib.request.urlopen(url).read()
        data = response.decode('utf-8')
        return data
         
    def store(data,file):
        f = open(file,"w+")
        f.write(data)
        f.close()
        print('wrote file: ' + file)

    urls = ['http://student.mit.edu/catalog/m1a.html',
            'http://student.mit.edu/catalog/m1b.html',
            'http://student.mit.edu/catalog/m1c.html',
            'http://student.mit.edu/catalog/m2a.html',
            'http://student.mit.edu/catalog/m2b.html',
            'http://student.mit.edu/catalog/m2c.html',
            'http://student.mit.edu/catalog/m3a.html',
            'http://student.mit.edu/catalog/m3b.html',
            'http://student.mit.edu/catalog/m4a.html',
            'http://student.mit.edu/catalog/m4b.html',
            'http://student.mit.edu/catalog/m4c.html',
            'http://student.mit.edu/catalog/m4d.html',
            'http://student.mit.edu/catalog/m4e.html',
            'http://student.mit.edu/catalog/m4f.html',
            'http://student.mit.edu/catalog/m4g.html',
            'http://student.mit.edu/catalog/m5a.html',
            'http://student.mit.edu/catalog/m5b.html',
            'http://student.mit.edu/catalog/m6a.html',
            'http://student.mit.edu/catalog/m6b.html',
            'http://student.mit.edu/catalog/m6c.html',
            'http://student.mit.edu/catalog/m7a.html',
            'http://student.mit.edu/catalog/m8a.html',
            'http://student.mit.edu/catalog/m8b.html',
            'http://student.mit.edu/catalog/m9a.html',
            'http://student.mit.edu/catalog/m9b.html',
            'http://student.mit.edu/catalog/m10a.html',
            'http://student.mit.edu/catalog/m10b.html',
            'http://student.mit.edu/catalog/m11a.html',
            'http://student.mit.edu/catalog/m11b.html',
            'http://student.mit.edu/catalog/m11c.html',
            'http://student.mit.edu/catalog/m12a.html',
            'http://student.mit.edu/catalog/m12b.html',
            'http://student.mit.edu/catalog/m12c.html',
            'http://student.mit.edu/catalog/m14a.html',
            'http://student.mit.edu/catalog/m14b.html',
            'http://student.mit.edu/catalog/m15a.html',
            'http://student.mit.edu/catalog/m15b.html',
            'http://student.mit.edu/catalog/m15c.html',
            'http://student.mit.edu/catalog/m16a.html',
            'http://student.mit.edu/catalog/m16b.html',
            'http://student.mit.edu/catalog/m18a.html',
            'http://student.mit.edu/catalog/m18b.html',
            'http://student.mit.edu/catalog/m20a.html',
            'http://student.mit.edu/catalog/m22a.html',
            'http://student.mit.edu/catalog/m22b.html',
            'http://student.mit.edu/catalog/m22c.html']

    for url in urls:
        index = url.rfind('/') + 1
        data = pull(url)
        file = url[index:]
        store(data,file)

        print('pulled: ' + file)
        print('--- waiting ---')
        time.sleep(15)

```

I saw this as an opportunity to be more efficient with my code by creating a function that does this for all of the required tabs in the Excel spreadsheet, as shown below:



The function is then used in the code below to efficiently output a single Data Frame with all of the data:

```python
list_df = []
years = []
year = 2020
while year >= 1992:
    years.append(str(year))
    year -= 1

for year in years:
    add_dataframe(list_df, year)
    
df_stacked = pd.concat(list_df)
df_stacked.reset_index(inplace=True)
df_stacked.drop("index", axis=1, inplace=True)
```

I created a separate YAML file that contained the credentials of the database connection that had to be established to load the data into a designated table. The benefit of this type of file is that it stores this sensitive information in a separate file so that it is not hard-coded in the main file.

Once the connection was made, I created the table and loaded the records from the Data Frame into the table, as shown below:

```python
# Secure Connection to the Database
db = yaml.safe_load(open("db.yaml"))
config = {
    "user":     db["user"],
    "password": db["pwrd"],
    "host":     db["host"],
    "auth_plugin":  "mysql_native_password"
}
cnx = mysql.connector.connect(**config)

MyCursor=cnx.cursor()
queries = []

# Creating the database and the table `mrts`
queries.append("DROP DATABASE IF EXISTS mrtsdb")
queries.append("CREATE DATABASE mrtsdb")
queries.append("USE mrtsdb")
queries.append("""CREATE TABLE mrts (
    kind_of_business VARCHAR (300) NOT NULL,
    value FLOAT NOT NULL,
    period DATE NULL) ENGINE=InnoDB DEFAULT CHARSET=UTF8MB4 COLLATE=utf8mb4_0900_ai_ci""")

for query in queries:
    MyCursor.execute(query)

...

# Inserting each row of the DataFrame to the MySQL table and committing
for row_num in range(0, len(df_stacked)):
    row_data = df_stacked.iloc[row_num]
    value = (row_data[0], row_data[1], row_data[2])
    sql = "INSERT INTO mrts (kind_of_business, value, period) VALUES (%s, %s, %s)"
    MyCursor.execute(sql, value)

cnx.commit()

```

As a result, the data was effectively loaded into the database table, as shown below in Figure 1:

| ![download](https://github.com/ukthanki/MIT_MRTS_ETL/assets/42117481/ae5ff829-5165-419a-aa87-0663c492c8b0)| 
|:--:| 
| **Figure 1.** MRTS data loaded into the *mrts* table in MySQL. |

I was then able to execute various SELECT statements through Python to visualize the data to gain various insights. For example, by executing the following code below, I was able to plot the data using Matplotlib, as shown below in Figure 2:

```python
# (5) Retail and food services sales, total Yearly Trend
query5 = """
SELECT SUM(`value`), YEAR(period) FROM mrts WHERE kind_of_business = 'Retail and food services sales, total'
GROUP BY 2 ORDER BY period
"""
MyCursor.execute(query5)
month = []
sales = []
for row in MyCursor.fetchall():
    sales.append(row[0])
    month.append(row[1])
    
plt.plot(month, sales)
plt.title("Retail and food services sales, total - Yearly")
plt.xlabel("Year")
plt.ylabel("Sales (USD, Million)")
plt.show()
```

| ![download](https://github.com/ukthanki/MIT_MRTS_ETL/assets/42117481/0ee68c9d-b29c-4251-b37e-93067cfae930)| 
|:--:| 
| **Figure 2.** Sales vs. Year for Retail and Food Services. |

This project was quite insightful because it focused heavily on ETL and how it may be done programmatically as opposed to manually. I could have produced the same plots by performing all steps in Python only, but by loading the data in MySQL, it became available to a wider audience for querying and analysis; this represents a real-world situation as a result because data must be accessible easily by multiple entities.

**You can view the full Project in the "module_8.py" and "Module 8_Umang_Thanki.ipynb" files in the Repository.**

[Go to Repo](https://github.com/ukthanki/MIT_Sensemaking_Data_Pipeline_Project)

In this project, we extract unstructured date from the MIT Course Catalog, clean and process the data, and visualize it in a D3 Web Application.
