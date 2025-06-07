# NASA-Near-Earth-Object-NEO-Tracking-Insights-using-Public-API
This is a simple project that helps us track asteroids and comets that come close to Earth. It uses data from NASA's public API to show useful information about these space objects.

#What Does This Project Do?
Gets real data about asteroids that pass near Earth
Tells us their size, speed, and how close they came
Shows if they are dangerous or not
Lets you choose a date to see what was flying by on that day

#Tools Used
NASA API – free data from NASA
Python – to write the code
Requests – to get data from the internet
Pandas / Matplotlib (optional) – to show data in tables or charts

git clone https://github.com/lakskarthi/nasa-neo-tracker.git
cd nasa-neo-tracker

#Why this is so cool
Learn how to use real space data
Practice working with APIs and Python
Understand what’s flying close to our planet
Great beginner project for science + coding lovers 

GitHub: @Lakskarthi



!pip install requests
API_KEY="HIpHooCJcd7KVx1nbBGQCIIXH1PdVTly3eZkFToR"
import requests
url = f"https://api.nasa.gov/neo/rest/v1/feed?start_date=2024-01-01&end_date=2024-01-08&api_key={API_KEY}"
response = requests.get(url)
response

data = response.json()
data

asteriods_data=[]
target = 10000
url= f"https://api.nasa.gov/neo/rest/v1/feed?start_date=2024-01-01&end_date=2024-01-08&api_key={API_KEY}"

response=requests.get(url)
data=response.json()
details=data['near_earth_objects']
while len(asteriods_data) < target:
    for date,ast_details in details.items():
        for ast in ast_details:
            asteriods_data.append(dict(
                id=ast['id'],
                name=ast['name'],
                magnitude=ast['absolute_magnitude_h'],
                dia_min=ast['estimated_diameter']['kilometers']['estimated_diameter_min'],
                dia_max=ast['estimated_diameter']['kilometers']['estimated_diameter_max'],
                hazardous=ast['is_potentially_hazardous_asteroid']
                
            ))          
            if len(asteriods_data)==target:
                break
        if len(asteriods_data)==target:
                break
        url=data['links']['next']

asteriods_data=[]
target = 10000
url= f"https://api.nasa.gov/neo/rest/v1/feed?start_date=2024-01-01&end_date=2024-01-08&api_key={API_KEY}"

response=requests.get(url)
data=response.json()
details=data['near_earth_objects']
while len(asteriods_data) < target:
    for date, ast_details in details.items():
        for ast in ast_details:
            if 'close_approach_data' in ast and ast['close_approach_data']:
                approach_data = ast['close_approach_data'][0]

                asteriods_data.append(dict(
                    neo_id = ast['neo_reference_id'],
                    close_approach_data = approach_data['close_approach_date_full'],
                    epoch_close_date = approach_data['epoch_date_close_approach'],
                    velocity = approach_data['relative_velocity']['kilometers_per_second'],
                    distance = approach_data['miss_distance']['kilometers'],
                    astronomical_AU = approach_data['miss_distance']['astronomical'],
                    lunar = approach_data['miss_distance']['lunar'],
                    Orbit = approach_data['orbiting_body']
                ))

            if len(asteriods_data) == target:
                break
        if len(asteriods_data)==target:
            break
        url=data['links']['next']

        import pandas as pd
import pymysql
pd.DataFrame(asteriods_data).to_csv("asteroids_data.csv", index=False)
print("Data saved to asteroids_data.csv")

import streamlit as st
import pandas as pd
import pymysql

# Page title
st.title("NASA PROJECT")

# Database connection (if needed)
conn = pymysql.connect(host='localhost', user='root', password='laks', database='nasa')
cur = conn.cursor()
cur.execute("""CREATE TABLE Asteroids (id INT,
        name VARCHAR(255),
        magnitude FLOAT,
        dia_min FLOAT,
        dia_max FLOAT,
        hazardous BOOLEAN
    )
""")



cur.execute("""
    CREATE TABLE Close_Approaches (
        id INT,
        close_approach_date DATE,
        epoch_close_date DATETIME,
        velocity_km_s FLOAT,
        miss_distance_km FLOAT,
        miss_distance_lunar FLOAT,
        orbiting_body VARCHAR(50)
    )
""")

# Load CSV
df = pd.read_csv("asteroids_data.csv")
df['close_approach_date'] = pd.to_datetime(df['close_approach_date'])
df['month'] = df['close_approach_date'].dt.month

# Sidebar menu
selected = st.sidebar.selectbox("Main Menu", ["Home", "Queries"])

if selected == "Home":
    st.image("sky.jpg", caption="Ready to Explore", use_column_width=True)
    

elif selected == 'Queries':
    options = st.selectbox("Queries", [
        "1. Display the entire table",
        "2. Count how many times each asteroid has approached Earth",
        "3. Average velocity of each asteroid over multiple approaches",
        "4. List top 10 fastest asteroids",
        "5. Find potentially hazardous asteroids that have approached Earth more than 3 times",
        "6. Find the month with the most asteroid approaches",
        "7. Get the asteroid with the fastest ever approach speed",
        "8. Sort asteroids by maximum estimated diameter (descending)",
        "9. Asteroid whose closest approach is getting nearer over time",
        "10. Display name of each asteroid with date and miss distance of closest approach",
        "11. List asteroids with velocity > 50,000 km/h",
        "12. Count how many approaches happened per month",
        "13. Find asteroid with highest brightness (lowest magnitude)",
        "14. Number of hazardous vs non-hazardous asteroids",
        "15. Asteroids closer than the Moon (miss_distance_lunar < 1)",
        "16. Astronomical distance within 0.05 AU",
        "17. Number of asteroid approaches by day of the week",
        "18. Top 10 closest asteroid approaches (in km)",
        "19. Get the asteroid with the slowest ever approach speed",
        "20. Find the month with the least asteroid approaches"
        
    ], placeholder='Choose an option..')

    if options == "1. Display the entire table":
        st.dataframe(df)

    elif options == "2. Count how many times each asteroid has approached Earth":
        result = df['name'].value_counts().reset_index()
        result.columns = ['name', 'approach_count']
        st.dataframe(result)

    elif options == "3. Average velocity of each asteroid over multiple approaches":
        result = df.groupby('name')['velocity_km_s'].mean().reset_index()
        result.columns = ['name', 'avg_velocity_km_s']
        st.dataframe(result)

    elif options == "4. List top 10 fastest asteroids":
        result = df.groupby('name')['velocity_km_s'].max().reset_index()
        result = result.sort_values(by='velocity_km_s', ascending=False).head(10)
        st.dataframe(result)

    elif options == "5. Find potentially hazardous asteroids that have approached Earth more than 3 times":
        hazardous = df[df['hazardous'] == True]
        counts = hazardous['name'].value_counts()
        result = counts[counts > 3].reset_index()
        result.columns = ['name', 'approach_count']
        st.dataframe(result)

    elif options == "6. Find the month with the most asteroid approaches":
        most_common_month = df['month'].value_counts().idxmax()
        result = pd.DataFrame({'Most Frequent Month': [most_common_month]})
        st.dataframe(result)

    elif options == "7. Get the asteroid with the fastest ever approach speed":
        idx = df['velocity_km_s'].idxmax()
        result = df.loc[[idx], ['name', 'velocity_km_s']]
        st.dataframe(result)

    elif options == "8. Sort asteroids by maximum estimated diameter (descending)":
        st.warning("Column 'estimated_diameter_max' not available in the dataset.")
        
    elif options == "9. Asteroid whose closest approach is getting nearer over time":
        df_sorted = df.sort_values(['name', 'close_approach_date'])
        st.dataframe(df_sorted[['name', 'close_approach_date', 'miss_distance_km']])

    elif options == "10. Display name of each asteroid with date and miss distance of closest approach":
        idx = df.groupby('name')['miss_distance_km'].idxmin()
        result = df.loc[idx, ['name', 'close_approach_date', 'miss_distance_km']]
        st.dataframe(result)

    elif options == "11. List asteroids with velocity > 50,000 km/h":
        threshold_km_s = 50000 / 3600
        result = df[df['velocity_km_s'] > threshold_km_s]
        st.dataframe(result[['name', 'velocity_km_s', 'close_approach_date']])

    elif options == "12. Count how many approaches happened per month":
        counts = df['month'].value_counts().reset_index()
        counts.columns = ['month', 'approach_count']
        st.dataframe(counts)

    elif options == "13. Find asteroid with highest brightness (lowest magnitude)":
        st.warning("Brightness or magnitude data not available in the dataset.")

    elif options == "14. Number of hazardous vs non-hazardous asteroids":
        counts = df.groupby('hazardous')['name'].nunique().reset_index()
        counts.columns = ['is_hazardous', 'unique_asteroid_count']
        st.dataframe(counts)

    elif options == "15. Asteroids closer than the Moon (miss_distance_lunar < 1)":
        result = df[df['miss_distance_lunar'] < 1]
        st.dataframe(result[['name', 'close_approach_date', 'miss_distance_lunar']])

    elif options == "16. Astronomical distance within 0.05 AU":
        AU_LIMIT_KM = 0.05 * 149597870.7  # ≈ 7,479,893.535 km
        result = df[df['miss_distance_km'] < AU_LIMIT_KM]
        st.dataframe(result[['name', 'close_approach_date', 'miss_distance_km']])

    elif options == "17. Number of asteroid approaches by day of the week":
        df['day_of_week'] = df['close_approach_date'].dt.day_name()
        result = df['day_of_week'].value_counts().reset_index()
        result.columns = ['Day of Week', 'Approach Count']
        st.dataframe(result)

    elif options == "18. Top 10 closest asteroid approaches (in km)":
        result = df.sort_values(by='miss_distance_km').head(10)
        st.subheader("Top 10 Closest Asteroid Approaches")
        st.dataframe(result[['name', 'close_approach_date', 'miss_distance_km']])

    elif options == "19. Get the asteroid with the slowest ever approach speed":
        min_speed = df['velocity_km_s'].min()
        result = df[df['velocity_km_s'] == min_speed][['name', 'velocity_km_s', 'close_approach_date']]
        st.subheader("Asteroid with the Slowest Ever Approach Speed")
        st.dataframe(result)

    elif options == "20. Find the month with the least asteroid approaches":
        least_common_month = df['month'].value_counts().idxmin()
        result = pd.DataFrame({'Least Frequent Month': [least_common_month]})
        st.dataframe(result)
