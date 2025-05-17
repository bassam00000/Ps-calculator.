import streamlit as st
import pandas as pd
from datetime import datetime, timedelta

st.title("Fair Game Cost Calculator (Shared via Google Sheet)")

RATE_PER_HOUR = 85
RATE_PER_MIN = RATE_PER_HOUR / 60

st.markdown("### Live Player Times from Google Sheet")

try:
    # Google Sheet in CSV format
    csv_url = "https://docs.google.com/spreadsheets/d/10-lNuc3JEIJul2NXrFZffzukvy1mlNhwx2zXwpYm0zE/export?format=csv"
    df = pd.read_csv(csv_url)

    # Parse time columns
    df['start_time'] = pd.to_datetime(df['start_time'], format='%H:%M')
    df['end_time'] = pd.to_datetime(df['end_time'], format='%H:%M')

    players = df.to_dict('records')

    # Calculate global time range
    min_time = df['start_time'].min()
    max_time = df['end_time'].max()

    timeline = {}
    durations = {}
    current_time = min_time
    while current_time < max_time:
        present = [
            p['name']
            for p in players
            if p['start_time'] <= current_time < p['end_time']
        ]
        if present:
            cost_per_person = RATE_PER_MIN / len(present)
            for name in present:
                if name not in timeline:
                    timeline[name] = 0
                    durations[name] = 0
                timeline[name] += cost_per_person
                durations[name] += 1
        current_time += timedelta(minutes=1)

    total_cost = sum(timeline.values())

    st.markdown("### Results:")
    for person in timeline:
        time_minutes = durations[person]
        time_hours = round(time_minutes / 60, 2)
        cost = round(timeline[person], 2)
        st.write(f"**{person}**: Played for {time_hours} hours, Cost: {cost} EGP")

    st.success(f"Total Group Cost: {round(total_cost, 2)} EGP")

except Exception as e:
    st.error("Error reading data from Google Sheet.")
    st.exception(e)
