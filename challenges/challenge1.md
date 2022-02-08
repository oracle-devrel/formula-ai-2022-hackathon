# Introduction

Formula 1 is one of the most competitive sports in the world. Engineers and technicians from every team use weather radar screens, provided by Ubimet to the teams, which allows them to track the current weather and make predictions during the race. Race engineers relay precise information to drivers, including:
- How many minutes until it starts raining
- Intensity of the rain
- Which corner will be hit first by the rain
- Duration of the rain

Points, and even races sometimes, are won and lost based on making sense of what the weather is going to do during a race, and being prepared as a team to act accordingly.

Therefore, weather forecasting takes a big part on the possible outcome of a race.

Similarly, F1 2021, the official Formula 1 videogame developed by Codemasters, uses a physics engine that behaves like the real world.

# Competition

[Follow this link to create your notebook for the challenge.](https://www.kaggle.com/oracledevrel/formulaaihackathon2022) You're free to choose where to create a notebook that uses the dataset. When you register your submission in [the hackathon website](https://formulaaihack.com), make sure to link a URL to the notebook you've created in the submission, and any additional material like an instructional video / external design. 

Please also make sure you register your team and your submission via [the hackathon website](https://www.formulaaihack.com).

# Challenge
In challenge 1 of this hackathon, you will be presented with historical weather data from the RedBull Racing eSports team and you will be required develop an Artificial Intelligence model that is able to make accurate weather predictions / forecasts.

The data structure of each packet provided to you is as follows (example is a real packet from the dataset). You can have a detailed look at all structure definitions [in this file](https://github.com/jasperan/f1-telemetry-oracle/blob/main/telemetry_f1_2021/cleaned_packets.py). Note that in the dataset, two additional variables (not present in the game) have been added, which are:
- gamehost: it's an unique identifier to separate different dataset consumers. It's infrastructure-related and it's a dummy variable from a qualitative perspective. 
- timestamp: UNIX timestamp when the packet was emitted.

```python
class PacketSessionData(Packet):
    _fields_ = [
        ("header", PacketHeader),  # Header
        ("weather", ctypes.c_uint8),
        ("timestamp", unix_timestamp), # timestamp for when the packet was received
        ("gamehost", gamehost_name), # unique identifier for the host that captured the data (not relevant) 
        # Weather - 0 = clear, 1 = light cloud, 2 = overcast
        # 3 = light rain, 4 = heavy rain, 5 = storm
        ("track_temperature", ctypes.c_int8),  # Track temp. in degrees celsius
        ("air_temperature", ctypes.c_int8),  # Air temp. in degrees celsius
        ("total_laps", ctypes.c_uint8),  # Total number of laps in this race
        ("track_length", ctypes.c_uint16),  # Track length in metres
        ("session_type", ctypes.c_uint8),
        # 0 = unknown, 1 = P1, 2 = P2, 3 = P3, 4 = Short P
        # 5 = Q1, 6 = Q2, 7 = Q3, 8 = Short Q, 9 = OSQ
        # 10 = R, 11 = R2, 12 = R3, 13 = Time Trial
        ("track_id", ctypes.c_int8),  # -1 for unknown, 0-21 for tracks, see appendix
        ("formula", ctypes.c_uint8),
        # Formula, 0 = F1 Modern, 1 = F1 Classic, 2 = F2,
        # 3 = F1 Generic
        ("session_time_left", ctypes.c_uint16),  # Time left in session in seconds
        ("session_duration", ctypes.c_uint16),  # Session duration in seconds
        ("pit_speed_limit", ctypes.c_uint8),  # Pit speed limit in kilometres per hour
        ("game_paused", ctypes.c_uint8),  # Whether the game is paused
        ("is_spectating", ctypes.c_uint8),  # Whether the player is spectating
        ("spectator_car_index", ctypes.c_uint8),  # Index of the car being spectated
        ("sli_pro_native_support", ctypes.c_uint8),
        # SLI Pro support, 0 = inactive, 1 = active
        ("num_marshal_zones", ctypes.c_uint8),  # Number of marshal zones to follow
        ("marshal_zones", MarshalZone * 21),  # List of marshal zones – max 21
        ("safety_car_status", ctypes.c_uint8),  # 0 = no safety car, 1 = full
        # 2 = virtual, 3 = formation lap
        ("network_game", ctypes.c_uint8),  # 0 = offline, 1 = online
        ("num_weather_forecast_samples", ctypes.c_uint8),
        # Number of weather samples to follow
        ("weather_forecast_samples", WeatherForecastSample * 56),
        # Array of weather forecast samples
        ("forecast_accuracy", ctypes.c_uint8),  # 0 = Perfect, 1 = Approximate.
        # The accuracy is a configurable in-game setting. We're providing data for both cases, although more for the 'perfect' mode.
        ("ai_difficulty", ctypes.c_uint8),  # AI Difficulty rating – 0-110
        ("season_link_identifier", ctypes.c_uint32),
        # Identifier for season - persists across saves
        ("weekend_link_identifier", ctypes.c_uint32),
        # Identifier for weekend - persists across saves
        ("session_link_identifier", ctypes.c_uint32),
        # Identifier for session - persists across saves
        ("pit_stop_window_ideal_lap", ctypes.c_uint8),
        # Ideal lap to pit on for current strategy (player)
        ("pit_stop_window_latest_lap", ctypes.c_uint8),
        # Latest lap to pit on for current strategy (player)
        ("pit_stop_rejoin_position", ctypes.c_uint8),
        # Predicted position to rejoin at (player)
        ("steering_assist", ctypes.c_uint8),  # 0 = off, 1 = on
        ("braking_assist", ctypes.c_uint8),  # 0 = off, 1 = low, 2 = medium, 3 = high
        ("gearbox_assist", ctypes.c_uint8),
        # 1 = manual, 2 = manual & suggested gear, 3 = auto
        ("pit_assist", ctypes.c_uint8),  # 0 = off, 1 = on
        ("pit_release_assist", ctypes.c_uint8),  # 0 = off, 1 = on
        ("ersassist", ctypes.c_uint8),  # 0 = off, 1 = on
        ("drsassist", ctypes.c_uint8),  # 0 = off, 1 = on
        ("dynamic_racing_line", ctypes.c_uint8),
        # 0 = off, 1 = corners only, 2 = full
        ("dynamic_racing_line_type", ctypes.c_uint8),  # 0 = 2D, 1 = 3D
    ]
``` 

Where the structure __header__ is a PacketHeader, which in turn is represented as:

```python
class PacketHeader(Packet):
    _fields_ = [
        ("packet_format", ctypes.c_uint16),  # 2021
        ("game_major_version", ctypes.c_uint8),  # Game major version - "X.00"
        ("game_minor_version", ctypes.c_uint8),  # Game minor version - "1.XX"
        ("packet_version", ctypes.c_uint8),
        # Version of this packet type, all start from 1
        ("packet_id", ctypes.c_uint8),  # Identifier for the packet type, see below
        ("session_uid", ctypes.c_uint64),  # Unique identifier for the session
        ("session_time", ctypes.c_float),  # Session timestamp
        ("frame_identifier", ctypes.c_uint32),
        # Identifier for the frame the data was retrieved on
        ("player_car_index", ctypes.c_uint8),  # Index of player's car in the array
        ("secondary_player_car_index", ctypes.c_uint8),
        # Index of secondary player's car in the array (splitscreen)
        # 255 if no second player
    ]
```

The __weather_forecast_samples__ being a WeatherForecastSample:

```python
class WeatherForecastSample(Packet):
    _fields_ = [
        ("session_type", ctypes.c_uint8),
        # 0 = unknown, 1 = P1, 2 = P2, 3 = P3, 4 = Short P, 5 = Q1
        # 6 = Q2, 7 = Q3, 8 = Short Q, 9 = OSQ, 10 = R, 11 = R2
        # 12 = Time Trial
        ("time_offset", ctypes.c_uint8),  # Time in minutes the forecast is for
        ("weather", ctypes.c_uint8),
        # Weather - 0 = clear, 1 = light cloud, 2 = overcast
        # 3 = light rain, 4 = heavy rain, 5 = storm
        ("track_temperature", ctypes.c_int8),  # Track temp. in degrees Celsius
        ("track_temperature_change", ctypes.c_int8),
        # Track temp. change – 0 = up, 1 = down, 2 = no change
        ("air_temperature", ctypes.c_int8),  # Air temp. in degrees celsius
        ("air_temperature_change", ctypes.c_int8),
        # Air temp. change – 0 = up, 1 = down, 2 = no change
        ("rain_percentage", ctypes.c_uint8),  # Rain percentage (0-100)
    ]
```

And the __marshal_zones__ being a MarshalZone:

```python
class MarshalZone(Packet):
    _fields_ = [
        ("zone_start", ctypes.c_float),
        # Fraction (0..1) of way through the lap the marshal zone starts
        ("zone_flag", ctypes.c_int8),
        # -1 = invalid/unknown, 0 = none, 1 = green, 2 = blue, 3 = yellow, 4 = red
    ]
```

# Data Set Notes
Considerations about the dataset:
- You'll be given the dataset in JSON and CSV formats. You're free to experiment with this non-relational data structure and develop your solution with this structure, or switch to the relational / tabular equivalent.
- It is recommended to filter out rows that provide no value to the AI model. This can occur if the number of forecast samples is 0 (num_weather_forecast_samples=0), or if the session type is 0 (unknown session type, as seen in the WeatherForecastSample definition above). This can be done doing an Exploratory Data Analysis.
- It contains one or many weather forecast samples, up to 56. This depends on the duration of the race / grand prix where the data was recorded. For example, if we play a 1 hour game, we will encounter an array of 6 elements; the first one, for the current time (not a forecast), and subsequent ones for the following timeframes: 5 minutes, 10, 15, 30, and 60. So, we'll have the ground truth (0-minute time_offset), and the corresponding forecasts. A packet received 5 minutes later will then be in the time_offset=0 and will represent the ground truth.
- The number of weather forecast samples for the same packet will be skewed, as not all games played in the dataset have the same length (and many have a lower length than 60 minutes). Normalization is recommended. The variables chosen to normalize / leave out of the model can be found by making an Exploratory Data Analysis.
- In case of a Grand Prix, where we have practice, qualifying and race sessions (equivalent to Friday, Saturday and Sunday in reality), the length of the WeatherForecastSample packet array will be longer, as it will contain more predictions for more race types. The maximum theoretical number of forecast samples for the same timestamp is 56. This means that you're able to access weather forecasting for the weekend, even if you're still exploring data from the qualifying or practice sessions, and take this data into consideration if you want.
- You have one packet per second. In reality, the game allows getting 2 PacketSessionData packets per second, but to avoid duplicates, we've removed half of them from the dataset.
- You are free to consider whichever variables you deem necessary to make an accurate prediction of the variable __weather__, together with a probability of an event happening (e.g. 85% cloudy).
- The unique index for the dataset is (session_uid, player_car_index, timestamp). Meaning that the granularity of the data will be as such. You can explore the total number of different sessions played by counting the session_uid distinct values. There may be several player_car_index variables for the session_uid (this may be the case in an online game where two players transmitting data into this dataset were playing in the same race). 
- For practice races (P1, P2, P3) the maximum session length is 1 hour.
- For qualifying races (Q1, Q2, Q3) the maximum session length is 18 minutes.


# Inputs and Outputs

Since this is a time-series-related model and the data structure is open for interpretation and decision of each one of the teams, **submissions will solely be based on the notebook provided by each one of the teams** (and everything contained inside the notebooks).

As inputs, each team is free to use as many variables to train the model. Please refer to the Scoring section down below to learn more about how each one of the notebooks will be scored.
Therefore, you may choose your model to have as many inputs as you want from the variables included in the dataset. In order for the judges to test and score your notebooks easily, please adhere to the standards for code readability and presentation (section explaining this further in the instructions).

## Output
The output format **must** be a dictionary or similar representation indicating the predicted weather type at 5, 10, 15, 30 and 60 minutes after a timestamp, and the rain percentage probability at that time:

```python
{
  '5':{
    'type': 3,
    'rain_percentage': 0.89 # e.g. 89% precipitation probability
  },
  '10':{
    'type': 3,
    '3': 0.64
  }
  '15':{
    'type': 3,
    '3': 0.52
  }
  '30':{
    'type': 2, # overcast
    '2': 0.19 # after 30 minutes, the weather type changes to overcast with a higher cumulative probability
  }
  '60':{
    'type': 3,
    '3': 0.94 # after 60 minutes, predictions suggest that we'll have light rain again with a 94% probability.
  }
} 
```

As you can see, in this minimal output, the "best" choice is expected for each one of the time frames (meaning the weather event most likely to happen and the chance of raining). As an explanation, in the above output, we have an 89% of rain probability and a predicted weather of "light rain" in 5 minutes; a 64% of rain and a predicted weather of "light rain" in 10 minutes; a 52% probability of rain and a predicted weather of "light rain" in 15 minutes; a 29% probability of rain and a predicted weather of "overcast" in 30 minutes; and a 94% rain probability and a weather of "light rain" in 60 minutes. We can conclude this example was pretty rainy.

Note that the weather type and the rain probability percentage are independent variables. In order to see if they're related, using modules to print correlation coefficients between the variables is recommended.

You can find all weather types in [this file (already referenced above)](https://github.com/jasperan/f1-telemetry-oracle/blob/main/telemetry_f1_2021/cleaned_packets.py), however here's a reminder of the weather types to understand the above example better: 0 = clear, 1 = light cloud, 2 = overcast, 3 = light rain, 4 = heavy rain, 5 = storm.

As said before, an exploratory data analysis is recommended to watch for feature importances and feature correlations when including/excluding variables from the dataset.

# Scoring

Due to the nature of the challenge, it will have its own weighting. Therefore, for this challenge, scoring will be divided into a technical and a non-technical part.

## Technical Scoring
The following things will be taken into consideration when doing a technical evaluation of challenge 1:
1. (5 points) Problem Modeling and approach (presented in the notebook)
2. (3 points) Providing a test code portion implementing your model, which includes exporting the model, importing it and using it with an example data point. More on this on the following sub-section: model exporting and testing. (presented in the notebook)
3. From the top 15 submissions, +2 extra points will be given to the top 5 teams with the highest model accuracy. The metric used will be Categorization Accuracy or Mean Absolute Error. Teams **must** indicate and print the accuracy of their model predictions in the notebook, using any available library, e.g. sklearn's accuracy score (meausrement must be MAE or Categorization Accuracy).

Therefore, the technical score will be out of 8 points, with the five top teams in the top 15 receiving +2 extra points. This means the maximum theoretical score is 10 points.

## Non-Technical Scoring
1. (3 points) Alignment to the problem: how well you've aligned your solution to the problem description.
2. (3 points) Quality of design: how well your design is constructed. This includes looking at the presentation of the design, including code presentation and readability (presented in the notebook)
3. (3 points) Originality of the design: how original your solution is perceived to be from the challenge's point of view. Taking an interesting approach, thinking outside the box, and testing assumptions count towards the originality of the design
4. (3 points) Video submission of the team explaining the solution: a video of your team members explaining the solution and your design. Points will be awarded considering how well it's presented, and how well the ideas are explained in the video.

The non-technical score will sum up to 12 points.

Both technical and non-technical scores will be summed in the end to decide the leaderboard.

### Problem Modeling and Approach

Provide a description on how you're approaching the problem, the variables that you're going to consider and why. We recommend doing this inside the code as you continue development, that way you'll have things clearer when solving the problem.

### Code Presentation and Readability

Clean code is code that is easy to understand and easy to change.

It's writing code that is:
- Readable
- Understandable
- Maintainable
- Extendable

We recommend adhering to the [PEP 8 -- Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/#naming-conventions), but if the code is readable and easily understandable we will accept it as well as any other code adhering to the standard.

### Model Accuracy 

We will rank the model accuracy and compare it with the rest of participants. Since model training and tuning is an important part of Data Science, we're awarding 2 additional points to the five teams present in the top 15 with the highest model accuracy.

In your code, you **must print** either a leaderboard (if you've trained several models with an AutoML tool) or scoring metrics for the models that you have, so that judges can check the accuracy individually. 

### Model Exporting and Testing

You must provide a portion of your code in which you test your own implemented ML model. You should follow a similar process to this:

```python
# load the model from disk
filename = 'trained_model.pkl'

loaded_model = pickle.load(open(filename, 'rb'))
loaded_model.predict(test_data) # test_data must be one row of information as-is, meaning, as it originally appears in the dataset provided for training. 

print(iris_y_test)
```

In this process, you need to load the model (not necessary if you have the pipeline in-memory), get an example row of data from the initial dataset, feed it into the pipeline/model and make a prediction. The output must show the resulting prediction, and the meaning of this prediction.

# Appendix

### Index Definition

For your information, this is the index definition for the dataset, meaning that there can't be two documents in the JSON collection with the same properties:

```json
{
  "name": "idxweather",
  "indexNulls": true,
  "unique": true,
  "lax": false,
  "scalarRequired": false,
  "fields": [
    {
      "path": "m_header.m_session_uid",
      "dataType": "NUMBER",
      "maxLength": null,
      "order": "ASC"
    },
    {
      "path": "m_header.m_player_car_index",
      "dataType": "NUMBER",
      "maxLength": null,
      "order": "ASC"
    },
    {
      "path": "timestamp",
      "dataType": "NUMBER",
      "maxLength": null,
      "order": "ASC"
    }
  ],
  "type": "functional"
}
```

### Basic Notebook

In [this website](../notebooks/report.html) you will find a very simple data representation from the dataset. There you'll be able to look at a very early-on Exploratory Data Analysis, including metrics like mean, median, max and min, 95% and 5% ranges, interquartile range, standard deviation and variance, skew, sum and distinct values for each one of the variables in the dataset.

You can open the HTML file in any browser and you will see something like this:

![](../img/example_visualization.PNG?raw=true)


## License
Copyright (c) 2022 Oracle and/or its affiliates.

Licensed under the Universal Permissive License (UPL), Version 1.0.

See [LICENSE](LICENSE) for more details.

ORACLE AND ITS AFFILIATES DO NOT PROVIDE ANY WARRANTY WHATSOEVER, EXPRESS OR IMPLIED, FOR ANY SOFTWARE, MATERIAL OR CONTENT OF ANY KIND CONTAINED OR PRODUCED WITHIN THIS REPOSITORY, AND IN PARTICULAR SPECIFICALLY DISCLAIM ANY AND ALL IMPLIED WARRANTIES OF TITLE, NON-INFRINGEMENT, MERCHANTABILITY, AND FITNESS FOR A PARTICULAR PURPOSE. FURTHERMORE, ORACLE AND ITS AFFILIATES DO NOT REPRESENT THAT ANY CUSTOMARY SECURITY REVIEW HAS BEEN PERFORMED WITH RESPECT TO ANY SOFTWARE, MATERIAL OR CONTENT CONTAINED OR PRODUCED WITHIN THIS REPOSITORY. IN ADDITION, AND WITHOUT LIMITING THE FOREGOING, THIRD PARTIES MAY HAVE POSTED SOFTWARE, MATERIAL OR CONTENT TO THIS REPOSITORY WITHOUT ANY REVIEW. USE AT YOUR OWN RISK. 