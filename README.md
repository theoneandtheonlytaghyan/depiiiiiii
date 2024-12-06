
import pandas as pd
import plotly.express as px
import dash
from dash import dcc, html
from dash.dependencies import Input, Output

# Load the dataset
file_path = 'C:/Users/Abdulrahman/Downloads/country_wise_latest.csv'
df = pd.read_csv(file_path)

# Drop rows with any null values
df = df.dropna()

# Calculate active cases if not provided
if 'Active' not in df.columns:
    df['Active'] = df['Confirmed'] - df['Recovered'] - df['Deaths']

# Print the dataframe to ensure data is loaded correctly
print(df.head())

# Initialize the Dash app
app = dash.Dash(__name__)

# Define the layout of the app
app.layout = html.Div([
    html.H1("COVID-19 Dashboard"),
    dcc.Dropdown(
        id='country-dropdown',
        options=[{'label': country, 'value': country} for country in df['Country/Region'].unique()],
        value='United States',
        clearable=False
    ),
    dcc.Graph(id='covid-graph'),
    html.Div(id='debug-output')  # Add a debug output area
])

# Define the callback to update the graph
@app.callback(
    Output('covid-graph', 'figure'),
    Output('debug-output', 'children'),  # Add debug output
    [Input('country-dropdown', 'value')]
)
def update_graph(selected_country):
    # Filter data for the selected country
    country_data = df[df['Country/Region'] == selected_country]
    if country_data.empty:
        return {}, f"No data found for {selected_country}"

    # Create a line chart
    fig = px.line(country_data, x='Country/Region', y=['Confirmed', 'Recovered', 'Deaths', 'Active'],
                  labels={'value': 'Cases', 'variable': 'Category'},
                  title=f'COVID-19 Cases in {selected_country}')

    debug_message = f"Plotting data for {selected_country}. Data shape: {country_data.shape}"
    return fig, debug_message

# Run the app
if __name__ == '__main__':
    app.run_server(debug=True)
