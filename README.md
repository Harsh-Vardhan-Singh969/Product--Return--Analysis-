#import pandas as pd
import numpy as np
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
from datetime import datetime
import matplotlib.pyplot as plt
import seaborn as sns
from IPython.display import display, HTML

# ====================================
# 1. SUPER-SIZED SAMPLE DATA GENERATION
# ====================================
np.random.seed(42)
num_records = 1500  # 3x more data points

categories = ['Electronics', 'Clothing', 'Home', 'Beauty', 'Toys', 'Groceries']
reasons = ['Defective', 'Wrong Size', 'Changed Mind', 'Late Delivery', 
           'Damaged', 'Wrong Item', 'Not as Described']
regions = ['North', 'South', 'East', 'West', 'Central', 'International']

data = {
    'order_id': [f'ORD{10000+i}' for i in range(num_records)],
    'product_category': np.random.choice(categories, num_records, p=[0.25, 0.2, 0.15, 0.15, 0.15, 0.1]),
    'return_reason': np.random.choice(reasons, num_records, p=[0.2, 0.15, 0.15, 0.1, 0.2, 0.1, 0.1]),
    'return_date': pd.date_range('2023-01-01', periods=num_records, freq='H').tolist(),  # Hourly data
    'region': np.random.choice(regions, num_records),
    'refund_amount': np.round(np.random.lognormal(mean=3.5, sigma=0.8, size=num_records), 2),
    'customer_age': np.random.normal(loc=35, scale=10, size=num_records).astype(int),
    'product_rating': np.random.randint(1, 6, num_records)
}

df = pd.DataFrame(data)
df['return_month'] = df['return_date'].dt.month_name()
df['return_weekday'] = df['return_date'].dt.day_name()
df['return_hour'] = df['return_date'].dt.hour
df['processing_days'] = np.random.randint(1, 21, num_records)

# ===================================
# 2. JUMBO-SIZED INTERACTIVE VISUALS
# ===================================

# Configure global layout
WIDTH = 1400
HEIGHT = 900
FONT_SIZE = 16
BG_COLOR = '#f9f9f9'

# 1. MEGA TREEMAP (Category > Reason > Region)
fig1 = px.treemap(df, path=['product_category', 'return_reason', 'region'],
                 values='refund_amount',
                 color='refund_amount',
                 color_continuous_scale='Rainbow',
                 title='<b>RETURN VALUE DISTRIBUTION</b><br>Size=Refund Amount, Color=Refund Value',
                 width=WIDTH, height=HEIGHT)
fig1.update_layout(font_size=FONT_SIZE,
                 plot_bgcolor=BG_COLOR,
                 paper_bgcolor=BG_COLOR)

# 2. GIANT FACET GRID (Multiple Dimensions)
fig2 = px.scatter(df, x='processing_days', y='refund_amount',
                 color='product_category', size='product_rating',
                 facet_col='region', facet_row='return_reason',
                 title='<b>MULTIDIMENSIONAL RETURN ANALYSIS</b>',
                 width=WIDTH, height=HEIGHT*1.2,
                 labels={'processing_days': 'Processing Time (days)',
                        'refund_amount': 'Refund Amount ($)'})
fig2.update_layout(font_size=FONT_SIZE-2)

# 3. FULLSCREEN TIME SERIES HEATMAP
hourly_returns = df.groupby(['return_weekday', 'return_hour']).size().unstack()
fig3 = go.Figure(go.Heatmap(
    z=hourly_returns.values,
    x=hourly_returns.columns,
    y=hourly_returns.index,
    colorscale='Viridis',
    colorbar=dict(title='Return Count')
))
fig3.update_layout(
    title='<b>HOURLY RETURN PATTERNS BY WEEKDAY</b>',
    xaxis_title='Hour of Day',
    yaxis_title='Day of Week',
    width=WIDTH,
    height=HEIGHT,
    font_size=FONT_SIZE
)

# 4. EXTENDED DASHBOARD LAYOUT
app = HTML(f"""
<style>
    .report {{
        width: 95vw;
        margin: 0 auto;
        font-family: Arial;
        background: {BG_COLOR};
        padding: 20px;
    }}
    .header {{
        background: #2c3e50;
        color: white;
        padding: 20px;
        text-align: center;
        margin-bottom: 30px;
        border-radius: 10px;
    }}
    .plot-container {{
        background: white;
        padding: 20px;
        margin-bottom: 30px;
        border-radius: 10px;
        box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    }}
    h2 {{
        color: #3498db;
        border-bottom: 2px solid #3498db;
        padding-bottom: 10px;
    }}
</style>

<div class="report">
    <div class="header">
        <h1>📊 PRODUCT RETURN ANALYSIS DASHBOARD</h1>
        <p>Generated on {datetime.now().strftime('%Y-%m-%d %H:%M:%S')} | {num_records:,} Records Analyzed</p>
    </div>
    
    <div class="plot-container">
        <h2>💰 Value Distribution</h2>
        {fig1.to_html(full_html=False, include_plotlyjs='cdn')}
    </div>
    
    <div class="plot-container">
        <h2>📊 Multidimensional Analysis</h2>
        {fig2.to_html(full_html=False, include_plotlyjs='cdn')}
    </div>
    
    <div class="plot-container">
        <h2>⏱ Time Patterns</h2>
        {fig3.to_html(full_html=False, include_plotlyjs='cdn')}
    </div>
    
    <div class="plot-container">
        <h2>📝 Executive Summary</h2>
        <ul>
            <li>Total refund value: ${df['refund_amount'].sum():,.2f}</li>
            <li>Most returned category: {df['product_category'].value_counts().idxmax()}</li>
            <li>Peak return hour: {df['return_hour'].value_counts().idxmax()}:00</li>
            <li>Average processing time: {df['processing_days'].mean():.1f} days</li>
        </ul>
    </div>
</div>
""")

# ==============================
# 3. SAVE AS FULLSCREEN HTML
# ==============================
with open('fullscreen_return_analysis.html', 'w') as f:
    f.write(f"""
    <!DOCTYPE html>
    <html>
    <head>
        <title>Fullscreen Return Analysis</title>
        <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
        <style>
            body {{ 
                margin: 0; 
                padding: 0; 
                font-family: Arial; 
                background: {BG_COLOR};
            }}
            .plot {{
                width: 95vw;
                height: 95vh;
                margin: 20px auto;
            }}
        </style>
    </head>
    <body>
        {fig1.to_html(full_html=False, div_id='plot1')}
        {fig2.to_html(full_html=False, div_id='plot2')}
        {fig3.to_html(full_html=False, div_id='plot3')}
    </body>
    </html>
    """)

# Display in notebook (if using Colab/Jupyter)
display(app)

print(f"""
✅ ANALYSIS COMPLETE! Files generated:
1. fullscreen_return_analysis.html (All visuals in one scrollable page)
2. Data contains {len(df):,} records with {len(df.columns)} dimensions
""") Product--Return--Analysis-
