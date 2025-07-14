import streamlit as st
from jumia_scraper import get_top_selling_items
from predictive_model import predict_future_sales
import matplotlib.pyplot as plt
import pandas as pd
import base64

st.set_page_config(page_title="Jumia Uganda Dashboard", layout="wide")

st.title("🛒 Jumia Uganda - Top Selling Items Dashboard")
st.markdown("This dashboard shows current top-selling items on Jumia Uganda and predicts future sales.")

# Sidebar
st.sidebar.header("🔍 Options")
refresh = st.sidebar.button("🔄 Refresh Data")

# Date range selection
st.sidebar.subheader("📅 Select Date Range")
start_date = st.sidebar.date_input("Start Date", pd.to_datetime("2024-01-01"))
end_date = st.sidebar.date_input("End Date", pd.to_datetime("2024-12-31"))

# Stock level selection
st.sidebar.subheader("📦 Filter by Stock Level")
min_stock = st.sidebar.slider("Minimum Stock", 0, 1000, 0)
max_stock = st.sidebar.slider("Maximum Stock", 0, 1000, 1000)

# Load data
@st.cache_data(show_spinner=False)
def load_data():
    return get_top_selling_items()

if refresh:
    st.cache_data.clear()
    df = get_top_selling_items()
else:
    df = load_data()

# Simulate additional columns
df['date'] = pd.to_datetime('2024-06-01')  # Placeholder date
df['stock'] = 500  # Placeholder stock for all items

# Apply filters
df_filtered = df[(df['date'] >= pd.to_datetime(start_date)) & 
                 (df['date'] <= pd.to_datetime(end_date)) & 
                 (df['stock'] >= min_stock) & 
                 (df['stock'] <= max_stock)]

# Show current items
st.subheader("📊 Current Top Selling Items")
st.dataframe(df_filtered.drop(columns='date'), use_container_width=True)

# Alert for low stock items
low_stock_items = df_filtered[df_filtered['stock'] < 10]
if not low_stock_items.empty:
    st.warning("⚠️ Some items have stock levels below 10!")
    st.dataframe(low_stock_items[['name', 'stock']])

    # Embed alert sound using HTML and base64
    sound_url = "https://www.soundjay.com/buttons/sounds/beep-07.mp3"
    st.markdown(f"""
    <audio autoplay>
        <source src="{sound_url}" type="audio/mpeg">
        Your browser does not support the audio element.
    </audio>
    """, unsafe_allow_html=True)

# Predictions
st.subheader("🔮 Predicted Future Top Sellers")
predicted_df = predict_future_sales(df_filtered.drop(columns='date'))
st.dataframe(predicted_df, use_container_width=True)

# Visualization
st.subheader("📈 Future Sales Prediction (Top 10)")
fig, ax = plt.subplots(figsize=(10, 6))
ax.barh(predicted_df['name'][:10][::-1], predicted_df['future_sales'][:10][::-1], color='skyblue')
ax.set_xlabel("Predicted Sales")
ax.set_ylabel("Item Name")
ax.set_title("Top 10 Predicted Best-Selling Items")
st.pyplot(fig)

st.markdown("---")
st.caption("Made with ❤️ using Streamlit | Data from Jumia Uganda")
