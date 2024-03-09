# Author: Marshall Tentis
# Date: 03/07/2024
# CS 066: Weather Application

from dash import Dash, html, dcc
from dash.dependencies import Input, Output
import requests
import datetime
import plotly.express as px

# Obtains the data for our graph
# OpenWeatherMap API Endpoint. Read about this here: https://openweathermap.org/appid
API_ENDPOINT = "http://api.openweathermap.org/data/2.5/forecast"
API_KEY = "41a1a3edd5af27415a85b88296f1bf5d"  # Replace with your own API key

#Pick your favorite zip code
zip_code = "50311"

# API parameters
params = {
    'zip': f"{zip_code},us",  # Assuming US ZIP code for simplicity, modify for other countries
    'appid': API_KEY,
    'units': 'imperial'  # Get weather in Fahrenheit. Change to 'metric' for Celsius.
}

response = requests.get(API_ENDPOINT, params=params)
DATA = response.json()

# Check if the response contains the necessary data. 
# Sometimes the server isn't serving! That's part of life with APIs.
if "list" not in DATA:
    print("Couldn't fetch the weather details.")

#I make this intermediate variable because I might want to come back later and 
    #use only *some* of the forecast items I get in the response,
    #in which case relevant_forecasts will be a subset of DATA["list"].
relevant_forecasts = DATA["list"]



# If you're interested, take a look at how we are "rearranging" the response data to make it easier to use.
# If you're not interested, make sure you at least understand what the output looks like so that you're able to use it properly!
def format_forecast_data(relevant_forecasts):
    forecast_times = [datetime.datetime.utcfromtimestamp(forecast["dt"]) for forecast in relevant_forecasts]
    #You need to figure out what this should be, and make sure it works with the rest of your code, especially below.
    forecast_data = [x["main"] for x in relevant_forecasts]
    formatted_data = []

    for i in range(len(forecast_data)):
        formatted_data.append(forecast_data[i])
        formatted_data[i]["time"] = forecast_times[i]
    return formatted_data

#Very important to know how this data is structured.
forecast_data = format_forecast_data(relevant_forecasts)


# Now that we put our data in the right format, we're going to try to 
# make a webapge out of it using Dash and plotly!


#creates the initial figure 
fig = px.line(forecast_data, x="time", y="temp", title='Forecast')
fig2 = px.line(forecast_data, x="time", y="humidity", title='Humidity')
fig3 = px.line(forecast_data, x="time", y="feels_like", title='feels_like')

app = Dash(__name__)

app.layout = html.Div(children = [
    dcc.Markdown( # sets a heading above the dropdown and graph
        id = "title",
        children = "## Weather Forecast for " + zip_code
    ),

    dcc.Dropdown( # creates a dropdown to select values to display
        id = "measure_select_dropdown",
        options = ["temp", "humidity", "feels like"],
        value = 'choose', # sets default value to display
        multi = True #allows selection of multiple values
    ),

    dcc.Graph(
        id = 'graph'

    )
    #dcc.Graph( #displays the graph on the page
    #    id = "weather_line_graph",
    #    figure = fig
    #),

    #dcc.Graph( #displays humidity graph
    #    id = "humidity_line_graph",
    #    figure = fig2
    #),

    #dcc.Graph( #displays real feel graph
    #    id = "realfeel_line_graph",
    #    figure = fig3
    #)
])

app.layout = html.Div(children = [
    dcc.Markdown( # sets a heading above the dropdown and graph
        id = "title",
        children = "## Weather Forecast for " + zip_code
    ),

    dcc.Dropdown( # creates a dropdown to select values to display
        id = "measure_select_dropdown",
        options = ["temp", "humidity", "feels like"],
        value = 'choose', # sets default value to display
        multi = True #allows selection of multiple values
    ),

    dcc.Graph(
        id = 'graph'

    )
    #dcc.Graph( #displays the graph on the page
    #    id = "weather_line_graph",
    #    figure = fig
    #),

    #dcc.Graph( #displays humidity graph
    #    id = "humidity_line_graph",
    #    figure = fig2
    #),

    #dcc.Graph( #displays real feel graph
    #    id = "realfeel_line_graph",
    #    figure = fig3
    #)
])

@app.callback(
    Output("graph","figure"),
    Input("measure_select_dropdown","value"),
)
def update_weather_graph(chosen_graph): #takes arguments?
    if chosen_graph == "temp":
        graph = px.line(forecast_data, x="time", y="temp", title='Forecast')
    elif chosen_graph == "humidity":
        graph = px.line(forecast_data, x="time", y="humidity", title='Humidity')
    elif chosen_graph == "feels like":
        graph = px.line(forecast_data, x="time", y="feels_like", title='feels_like')
    return graph

if __name__ == '__main__': # starts the server
    app.run_server(debug=True)
