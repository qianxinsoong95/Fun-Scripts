import streamlit as st
import random
import datetime
import json
import os
import pandas as pd
import altair as alt

# --- File paths ---
OPTIONS_FILE = "lunch_options_with_theme.json"
RECORD_FILE = "lunch_record.json"
VOTE_HISTORY_FILE = "vote_history.json"

# --- Sidebar: Developer Note ---
st.sidebar.markdown(
    "<p style='font-size: 10pt; color: gray;'>"
    "🛠️ <strong>Developed by Qian Xin Soong</strong><br>"
    "🐞 Please report bugs to <a href='mailto:qsoong@micron.com'>qsoong@micron.com</a>"
    "</p>",
    unsafe_allow_html=True
)

# --- Page Configuration ---
st.set_page_config(
    page_title="Lunch Decision Dashboard",
    page_icon="🍽️",
    layout="wide",
    initial_sidebar_state="expanded"
)

# --- Load and Save JSON ---
def load_data(file_path, default_data):
    if os.path.exists(file_path):
        with open(file_path, "r") as f:
            return json.load(f)
    else:
        return default_data

def save_data(file_path, data):
    with open(file_path, "w") as f:
        json.dump(data, f, indent=2)

# --- Load data from disk ---
lunch_options = load_data(OPTIONS_FILE, [])
lunch_record = load_data(RECORD_FILE, [])
vote_history = load_data(VOTE_HISTORY_FILE, [])

# --- Initialize session state ---
if "suggested_spot" not in st.session_state:
    st.session_state.suggested_spot = None

# --- Title ---
st.title("🍽️ Lunch Decision Dashboard")

# --- Sidebar: Add new lunch option ---
st.sidebar.header("➕ Add Lunch Option")
with st.sidebar.form("add_option_form"):
    name = st.text_input("Restaurant Name")
    location = st.text_input("Location")
    diet = st.selectbox("Dietary Preference", ["Any", "Halal", "Non-Halal", "Vegetarian", "Vegan", "Gluten-Free"])
    theme = st.text_input("Theme")
    lat = st.text_input("Latitude")
    lon = st.text_input("Longitude")
    submitted = st.form_submit_button("Add Option")

    if submitted:
        if name and location and lat and lon and theme:
            try:
                lat_val = float(lat)
                lon_val = float(lon)
                new_option = {"name": name, "location": location, "diet": diet, "theme": theme, "votes": 0, "lat": lat_val, "lon": lon_val}
                if not any(opt["name"].lower() == name.lower() for opt in lunch_options):
                    lunch_options.append(new_option)
                    save_data(OPTIONS_FILE, lunch_options)
                    st.success(f"Added {name} to lunch options.")
                else:
                    st.warning(f"{name} is already in the list.")
            except ValueError:
                st.error("Latitude and Longitude must be valid numbers.")
        else:
            st.error("Please enter all fields including coordinates and theme.")

# --- Admin Panel ---
st.sidebar.header("🔐 Admin Panel")
admin_password = st.sidebar.text_input("Enter Admin Password", type="password")

if admin_password == "admin123":
    st.sidebar.subheader("🔄 Reset Voting System")
    if st.sidebar.button("🔄 Reset All Votes"):
        for option in lunch_options:
            option["votes"] = 0
        save_data(OPTIONS_FILE, lunch_options)
        vote_history = []
        save_data(VOTE_HISTORY_FILE, vote_history)
        st.sidebar.success("✅ All votes and history have been reset.")
elif admin_password:
    st.sidebar.error("❌ Incorrect password.")

# --- Layout: Two columns ---
main_col, suggestion_col = st.columns([3, 2])

# --- Main Column ---
with main_col:
    st.subheader("🔍 Filter & Suggest Lunch Spot")
    filter_location = st.selectbox("Filter by Location", ["Any"] + sorted(set(opt["location"] for opt in lunch_options)))
    filter_diet = st.selectbox("Filter by Dietary Preference", sorted(set(opt["diet"] for opt in lunch_options)))
    filter_theme = st.selectbox("Filter by Theme", ["Any"] + sorted(set(opt["theme"] for opt in lunch_options)))

    filtered_options = [
        opt for opt in lunch_options
        if (filter_location == "Any" or opt["location"] == filter_location) and
           (filter_diet == "Any" or opt["diet"] == filter_diet) and
           (filter_theme == "Any" or opt["theme"] == filter_theme)
    ]

    if st.button("🎲 Suggest Lunch Spot"):
        if filtered_options:
            suggestion = random.choice(filtered_options)
            st.session_state.suggested_spot = suggestion
            with st.container():
                st.markdown("### 🎯 Suggested Lunch Spot")
                st.markdown(f"**{suggestion['name']}**")
                st.write(f"📍 Location: {suggestion['location']}")
                st.write(f"🥗 Diet: {suggestion['diet']}")
                st.write(f"🎨 Theme: {suggestion['theme']}")
                lat = suggestion['lat']
                lon = suggestion['lon']
                maps_url = f"https://www.google.com/maps/dir/?api=1&destination={lat},{lon}"
                st.markdown(f"[🗺️ Get Directions]({maps_url})", unsafe_allow_html=True)
        else:
            st.warning("No matching lunch options found.")

    st.subheader("🍴 Today's Lunch Record")
    today = datetime.date.today().strftime("%Y-%m-%d")
    group_name = st.text_input("Enter Group Name")
    selected_place = st.selectbox("Where did this group go for lunch today?", options=[opt["name"] for opt in lunch_options], index=0)

    if st.button("📍 Record Group's Lunch"):
        if group_name:
            record_entry = {
                "date": today,
                "group": group_name,
                "place": selected_place
            }
            lunch_record.append(record_entry)
            save_data(RECORD_FILE, lunch_record)
            st.success(f"Recorded: {group_name} went to {selected_place} on {today}")
        else:
            st.warning("Please enter a group name.")

    st.markdown("### 📆 Past Lunch Records by Group")
    if lunch_record:
        df_records = pd.DataFrame(lunch_record)
        required_columns = {"date", "group", "place"}
        if required_columns.issubset(df_records.columns):
            grouped = df_records.groupby(['date', 'group'], as_index=False)['place'].agg(lambda x: ', '.join(x))
            for _, row in grouped.iterrows():
                st.write(f"📅 {row['date']} | 👥 **{row['group']}**: {row['place']}")
        else:
            st.warning("Missing required columns in lunch records.")
    else:
        st.info("No lunch records yet.")

    st.subheader("📊 Vote for Your Favorite")
    voter_name = st.text_input("Enter Your Name")
    vote_location = st.selectbox("Filter by Location (Voting)", ["Any"] + sorted(set(opt["location"] for opt in lunch_options)))
    vote_diet = st.selectbox("Filter by Dietary Preference (Voting)", sorted(set(opt["diet"] for opt in lunch_options)))
    vote_theme = st.selectbox("Filter by Theme (Voting)", ["Any"] + sorted(set(opt["theme"] for opt in lunch_options)))

    vote_filtered_options = [
        opt for opt in lunch_options
        if (vote_location == "Any" or opt["location"] == vote_location) and
           (vote_diet == "Any" or opt["diet"] == vote_diet) and
           (vote_theme == "Any" or opt["theme"] == vote_theme)
    ]

    with st.expander("📝 Vote List Here", expanded=False):
        for i, opt in enumerate(vote_filtered_options):
            with st.container():
                st.markdown(f"### {opt['name']}")
                st.write(f"📍 Location: {opt['location']}")
                st.write(f"🥗 Diet: {opt['diet']}")
                st.write(f"🎨 Theme: {opt['theme']}")
                st.write(f"👍 Votes: {opt['votes']}")
                if st.button(f"👍 Vote for {opt['name']}", key=f"vote_{i}"):
                    if voter_name.strip():
                        opt["votes"] += 1
                        save_data(OPTIONS_FILE, lunch_options)
                        vote_entry = {
                            "restaurant": opt["name"],
                            "voter": voter_name.strip(),
                            "timestamp": datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                        }
                        vote_history.append(vote_entry)
                        save_data(VOTE_HISTORY_FILE, vote_history)
                        st.success(f"Thanks {voter_name} for voting for {opt['name']}!")
                    else:
                        st.warning("Please enter your name before voting.")

    st.subheader("📋 Current Lunch Options")
    with st.expander("⚡ List of Restaurant", expanded=False):
        for opt in lunch_options:
            with st.expander(f"{opt['name']}", expanded=False):
                st.write(f"**Location:** {opt['location']}")
                st.write(f"**Dietary Preference:** {opt['diet']}")
                st.write(f"**Theme:** {opt['theme']}")
                st.write(f"**Votes:** {opt['votes']}")

    st.markdown("### 📊 Voting Trends")
    df_votes = pd.DataFrame(lunch_options)
    if "votes" not in df_votes.columns:
        st.info("No vote data available.")
    else:
        df_votes = df_votes[df_votes["votes"] > 0]
        if df_votes.empty:
            st.info("No votes yet to display in the chart.")
        else:
            top_10_restaurants = df_votes.sort_values(by="votes", ascending=False).head(10)
            chart = alt.Chart(top_10_restaurants).mark_bar().encode(
                x=alt.X('name', sort='-y', title='Restaurant Name'),
                y=alt.Y('votes', title='Vote Count'),
                color=alt.Color('name', title='Restaurant')
            ).configure_axisX(
                labelAngle=90,
                labelFontSize=10
            ).configure_axisY(
                labelFontSize=10
            ).configure_legend(
                labelFontSize=11,
                titleFontSize=12
            ).properties(
                title='Which Restaurants to go?',
                width=500,
                height=400
            )
            st.altair_chart(chart, use_container_width=True)

    st.subheader("📝 Vote History")
    if vote_history:
        df_history = pd.DataFrame(vote_history)
        st.dataframe(df_history.sort_values(by="timestamp", ascending=False), use_container_width=True)
    else:
        st.info("No votes recorded yet.")

# --- Suggestion Column ---
with suggestion_col:
    st.markdown("## 🤔 Suggestion")
    st.write("You Vote la, then see how")
    if lunch_options:
        scores = {opt['name']: opt['votes'] for opt in lunch_options}
        sorted_options = sorted(lunch_options, key=lambda x: scores.get(x['name'], 0), reverse=True)
        top_pick = sorted_options[0]
        st.success(f"Today's Top Pick: {top_pick['name']} ({top_pick['location']}, {top_pick['diet']}, {top_pick['theme']})")

        lat = top_pick['lat']
        lon = top_pick['lon']
        maps_url = f"https://www.google.com/maps/dir/?api=1&destination={lat},{lon}"
        st.markdown(f"[🗺️ Get Directions]({maps_url})", unsafe_allow_html=True)

        st.markdown("### 🗺️ Lunch Location")
        if st.session_state.suggested_spot and "lat" in st.session_state.suggested_spot and "lon" in st.session_state.suggested_spot:
            map_data = pd.DataFrame([{
                "lat": st.session_state.suggested_spot["lat"],
                "lon": st.session_state.suggested_spot["lon"]
            }])
            st.map(map_data)
        else:
            default_map_data = pd.DataFrame([{"lat": 5.2189, "lon": 100.4491}])
            st.map(default_map_data)

        st.markdown("### 📈 Dashboard Stats")
        st.metric("Total Votes", sum(opt["votes"] for opt in lunch_options))
        st.metric("Lunch Records", len(lunch_record))
    else:
        st.info("Add lunch options to get smart suggestions.")
