# carvoyant_models

## Intro

The Carvoyant iOS app utilises tree ensemble regression models to predict carpark availability based on time, day of week, school term status, and weather conditions.

The app exclusively calculates availability for the current and next day, so all features except time are calculated (or downloaded in the case of weather) by the app. A future update will include a sandbox tab allowing the user to specify the day of week and weather conditions themselves. 

## Data collection

Government of Jersey publishes the live data of Green St, Minden Pl, Patriotic St, Sand St, Pier Rd, and Les Jardin carparks to https://www.gov.je/Travel/Motoring/Parking/pages/carparkspaces.aspx, which, on brief inspection of the source, gets its data from the public API https://sojpublicdata.blob.core.windows.net/sojpublicdata/carpark-data.json.

Since only two months of data have been collected so far, seasonal trends cannot be picked up yet, and this impact can be seen for Patriotic Street, which has trended upwards in availability each week so far. 

Bank holidays are accounted for as there have been two similarly behaving instances in the dataset already, and 6 small additional models are already being used in the app. Major Jersey holidays such as Liberation Day and the Battle of Flowers will be unpredictable this year, but the data gathered from 2024 will allow insight into each holiday in 2025 and onwards.

## Usage in Xcode project / app playground

Drag each .mlmodel file into the sidebar, ensuring they're added to the target. On build, a class for each model will be automatically generated. You can then make a simple call to the model using the built-in input class.

```
import CoreML

let model = try GREENST(configuration: MLModelConfiguration())

let input = GREENSTInput(
    IsHalfTerm: Int64(0), // not half term
    Precipitation: 0.12, // 0.12 mm/h rain fall
    WindSpeed: 23.57, // 23.57 km/h wind speed
    DayOfWeek: Int64(6), // sunday
    MinutesPastMidnight: Int64(980), // 4:20 pm
    WeekdayWeekend: Int64(0) // weekend
)

let result = try model.prediction(input: input)
print(Int(result.Available_Spaces))
```

For an app playground, which doesn't natively support .mlmodel resources and therefore won't generate the model class, use a dummy Xcode project to build and generate the class, then copy it into your app playground. Change the urlOfModelInThisBundle class to this in order to get it working on the iPad Playground app:
```
class var urlOfModelInThisBundle : URL {
  let resPath = Bundle(for: self).url(forResource: "GREENST", withExtension: "mlmodel")!
  return try! MLModel.compileModel(at: resPath)
}
```

## Paramaters

IsHalfTerm - 1 for inside half term, 0 for outside

Precipitation - hourly rain fall in mm

WindSpeed - hourly wind speed in km

DayOfWeek - 0 for monday through 6 for sundayn

MinutesPastMidnight - the time in minutes

WeekdayWeekend - 1 for weekday, 0 for weekend
