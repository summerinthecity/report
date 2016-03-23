# Summer in the City

## Introduction

The project aim is develop a novel prototype hourly weather forecasting for human thermal comfort in urban areas at street level.  The traditional weather forecast, as illustrated below, is based on the large scale weather system of high- and low- pressure systems.  Although weather is difficult to predict accurately, the forecasts have become more and more reliable.  They are now reliable enough to plan an outdoor trip (or to decide to stay at home) a week in advance.

![Traditional weather forecast](https://raw.githubusercontent.com/summerinthecity/report/master/images/project_oldforecast.jpg  "Traditional weather forecast")

However, the forecasts are not very accurate: they give a single temperature for a region the size of a province. The actual (local) temperature can be quite different from the forecast, depending on whether you are in city, a park, or close to water.  Actually, the forecast is valid for the official weather stations, which are located in rural areas, away from the cities where most people live.  As the popularity of the rain-radar has shown, people love to have better information, like *when* and *where* it will rain.  This project tries to bring a similar precision to the temperature forecast: by taking into account the local information, forecast the temperature in urban areas at street level.


![Street level weather forecast](https://raw.githubusercontent.com/summerinthecity/report/master/images/project_newforecast.png "Street level weather forecast")


## Heat stress

A heat wave is a prolonged period with excessively hot weather.  In the Netherlands the KNMI defines it as a period of at least 5 days with temperatures above 25 degrees Celsius, of which at least 3 days the temperature exceeds 30 degrees.  The high temperatures during a heat wave are not just uncomfortable, they can be quite dangerous too.  See for instance [this](http://www.metoffice.gov.uk/education/teens/case-studies/heatwave) for a description of the 2003 heatwave hitting Europe causing an estimated 20.000 deaths.  And even in the Netherlands a heat wave can be lethal.  Less dramatic, but economically as least as important, are the physical and psychological exhaustion caused by long summer nights when the temperature does not seem to drop at all.  The temperature is affected by the local conditions. Urban areas with trees or water are cooler than a city filled with concrete and asphalt.


![UHI Profile](https://raw.githubusercontent.com/summerinthecity/report/master/images/project_uhi_profile.png "The temperature is affected by the local conditions. Urban areas with trees or water are cooler than a city filled with concrete and asphalt.")



## Urban heat island

Cities are more susceptible to high temperatures than the country side.
This is caused in part by the absence of water, trees and vegetation, and reduced wind speeds. Another factor is typical architecture in the city: Gray concrete and black asphalt absorb most of the sunlight during daytime. With high-rise buildings on both sides, streets are practically sheltered canyons that trap the warmth. And air-conditioning cools the *inside* of the buildings, but dump all the heat outside. All this combines to increased temperatures in the city, sometimes several degrees warmer!


![Your local neighborhood](https://raw.githubusercontent.com/summerinthecity/report/master/images/project_buurt.jpg  "The last step is to further downscale the weather forecast to neighbourhood and street level.")


## Forecasts

The urban heat island effect is reasonably well reproduced by current numerical weather forecasting systems.  This does require, however, that the model uses novel and advanced parametrizations to represent cities and urbanization.  Also, a sufficiently high resolution (sub kilometer scale) combined with high quality geographical data is needed.  This poses demanding computational and data challenges.

In this project we will use WRF.  This model has recently been extended for urban applications, and there is a lot of experience with this model at Wageningen University.  New insights and experience will be used to further improve the model when needed.  However, to reach a street level forecast an extra effort could be required.  Statistical downscaling, computer learning techniques, or the development of a new model could be necessary.


![Special cargo bikes](https://raw.githubusercontent.com/summerinthecity/report/master/images/project_fiets.jpg  "Temperature, radiation, wind measurements are performed in-situ using a specially equipped bike.")

## Observations

Accurate observations are absolutely necessary to perform, validate, and improve weather forecasts.  Unfortunately, they rarely available at the right spatial and temporal resolution.  In this project, automated measurements at selected locations in Wageningen, and later possibly Amsterdam, are performed.  Also, a specially equipped bicycle, built and designed at Wageningen University, will be used for special measurement campaigns.  This will provide us with unique data, allowing us new insights in urban temperatures.

Alternative sources of observations will be investigated as well.  For instance via the Weather Underground project, a cooperation of amateur meteorologists.  Weather Underground has developed the world's largest network of personal weather stations (almost 23,000 stations in the US and over 13,000 across the rest of the world) that provides our site's users with the most localized weather conditions available. See [the weather underground website](http://www.wunderground.com/). But meteorological data could be derived from less obvious sources like the GSM cell phone network, or traffic control system.


![Crowdsourced temperature observations](https://raw.githubusercontent.com/summerinthecity/report/master/images/project_wunderground.png "Weather Underground has developed the world's largest network of personal weather stations, almost 23,000 stations in the US and over 13,000 across the rest of the world, that provides our site's users with the most localized weather conditions available. See http://www.wunderground.com/")


## Presentation

Finally, in this project we will try to make the developed forecasts generally available.  This could be in the form of maps showing thermal comfort forecasts, or a heat-wave alert.  The best medium is unclear yet, but could be a website, social media, or a mobile app.  

