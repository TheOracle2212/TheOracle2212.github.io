---
title:  "A Vacuum fluorescent display clock #02 "
mathjax: false
layout: post
categories: Clock, VFD, Arduino, Programming, GPS
---
## Alright!!
I got a Rasberry Pi Pico (not a W - decided I didn't need it - a potentially bad decision??). This meant that I had to port everything (all my code) to Earl E Philhower's library/ SDK port of Pi Pico. 
It wasn't too bad at all and I recall managing everything to be fine by the 2nd upload. 

In any case:

Using SoftwareSerial is a nuisance, high overheads, lousy bitbanging, and lots of computational power. The Pi Pico has 2 Serials, so why not put them to good use 

##!Pictures coming soon (I promise!)

If you recall, the Vacuum Florescent Display uses Serial by itself so that became Serial1, and Serial2 was for the GPS - wonderful!
```
void setup() {
  Serial1.setRX(17);
  Serial1.setTX(16);
  Serial1.begin(19200); // Start serial communication at 19200 baud

  Serial2.setRX(5);     // Set RX pin to GP5
  Serial2.setTX(4);     // Set TX pin to GP4
  Serial2.begin(9600);  // Start Serial1 at 9600 baud
}
```
I f
Unfortunately, the GPS wasn't as good as I hoped. Even in a surbaban area, I couldn't get some consistent form of time. It was very spotty, it took too long to start and 
gave me lots of infomation that just wasn't useful here. I only needed the time an the date (again - using the TinyGPS library).
In any case, I thought I might build the rest of the application and try to make use of the Screen Real estate. 

This led to a Sunrise/Sunset calculator and and interesting programming challenge to design a day of the week calculator. Oddly enough, its much harder than you think. 
```
 if (recalc == 100){ // Recalculation algorithm. Reduces the strain on the mcu. Can potentially decrease the number to 50 if you have the computational space.
    latitude = gps.location.lat();
    longitude = gps.location.lat();
    calcSunriseSunset(gps.date.year(), gps.date.month(), gps.date.day() , latitude, longitude, transit, sunrise, sunset);
    day_of_week = getDayOfWeek(localtime.day,localtime.month,localtime.year);
    Serial1.write(0x1B);
    Serial1.write(0xFC);
    recalc = 0;
  }
  ```
I wrote it in a routine that occurs once every so often becuase there's no point in recalculating it like every time the void main runs. It runs well enough that when the day changes, there's literally no lag at all; I guess it works fine.
Originally it ran every time the ```loop()``` ran, but I saw my poor Uno collapse under the weight, so I did this. I didn't really bother switching it back once I ported the code over to the Pi Pico becuase there was no need to. This just worked better.

This magic function converts the date to the day of the week:
```
char* getDayOfWeek(int day, int month, int year) { // Day of the week from the given date
    char* days[] = {"Sat", "Sun", "Mon", "Tue", "Wed", "Thu", "Fri"};

    if (month < 3) {  // Treat Jan and Feb as months 13 and 14 of previous year
        month += 12;
        year -= 1;
    }

    int K = year % 100;   // Year within the century
    int J = year / 100;   // Zero-based century

    int h = (day + (13 * (month + 1)) / 5 + K + (K / 4) + (J / 4) - (2 * J)) % 7;

    return days[(h + 7) % 7];  // Ensure positive index
} 
```
I'd originally written some 37 line monstrosity that did the job, albeit poorly and unoptimally, but this was a ChatGPT stroke of genius that converted that to this. I can only see a slight resemblence to the original function so even I have trouble reading and
understanding it. It just works and that's enough.
For the date, I wanted something more ... sophisticated... so I wrote a function to convert individual digits to Roman Numerals. But that turned out pretty poorly, another 56 line terror. This time I used Claude to 
make sense of it and I got this:
```
// Roman Numerals. Should be good.
String intToRoman(int num) {
    struct Roman {
        int value;
        const char *numeral;
    };

    Roman romanMap[] = {
        {1000, "M"}, {900, "CM"}, {500, "D"}, {400, "CD"},
        {100, "C"}, {90, "XC"}, {50, "L"}, {40, "XL"},
        {10, "X"}, {9, "IX"}, {5, "V"}, {4, "IV"},
        {1, "I"}
    };

    String roman = "";
    for (const auto &r : romanMap) {
        while (num >= r.value) {
            roman += r.numeral;
            num -= r.value;
        }
    }
    return roman;
}
```
It works well, and I've already used it for other projects with Roman Numerals (coming soon hopefully). I tried reducing some of the overheads on the original Claude generated function, and I think I made some useful headway with it
and I haven't encountered any issues with it.

At the bottom of the display, I put a scrolling string of international times, partially inspired by my trip to Europe. I put times from 9 different cities, approximately spread out well but also to indicate some cool places relevant to me
and all my attempts of making it scroll ended in failure. This bit of code is more contextual than the others, so you'll have to read the whole program for it to make more sense, but hopefully, this suffices:
```
void scrollText() {
    int msgLength = message.length();
    String displayBuffer = "";
    
    for (int i = 0; i < windowSize; i++) {
        displayBuffer += message[(scrollIndex + i) % msgLength];
    }
    
    Serial1.println(displayBuffer); // Output the scrolled text
    
    scrollIndex = (scrollIndex + scrollStep) % msgLength; // Update index with wrap-around
}
```
ChatGPT wrote this cool function that basically took in a string and when I wanted to update it, I just called the ```scrollText()``` function. This was quite useful and I tweaked the timing to get the scrolling exactly right. Again, for this one, it makes more sense when you read the whole program.

Overall though, I'm most proud of the RTC sync feature. For those unaware, the RP2040 has a RTC built into it which is mostly good. It does the job with a slight mistime, becuase nothing is perfect for time generation other than an atomic clock (GPS). 
This solved my problem of getting spotty GNSS service (GNSS is actually the right term, GPS is just a specfic constellation). It was quite easy to implement with the right help ([Thanks Max and Earl](https://github.com/earlephilhower/arduino-pico/issues/2832)). 

I ended up writing this which I believe was the easiest way of implementing it. 
```
  if (gps.time.isUpdated() == true){
    datetime_t date_to_set = {
    .year = gps.date.year(),
    .month = gps.date.month(),
    .day = gps.date.day(),
    .hour = gps.time.hour(),
    .min = gps.time.minute(),
    .sec = gps.time.second()
  };
  rtc_set_datetime(&date_to_set);
  } 
  datetime_t cur_date;
  rtc_get_datetime(&cur_date);
  DateTime utc = {cur_date.year, cur_date.month, cur_date.day, cur_date.hour, cur_date.min, cur_date.sec}; // Dataset input for the timezone adjustment

  DateTime localtime = adjustTimeZone(utc.year, utc.month, utc.day, utc.hour, utc.minute, utc.second, time_zone);
```
I feel like lots of times, we end up throwing in features and functions into the program that work in one sense but end up breaking something else or causing some other timing issue. You dont realise is until you remove some of the code. You solve one problem, but create another 2. Apart from creating
optimal and efficient code - a statement of this project, trying not to break already existing stuff, it became harder. Please let me know if you've got any feedback for that routine (or any of the other routines - use my email address below).

In my next post I'll discuss the hardware and show my design considerations for it and the iterations I went through. I got a new Bambu Lab P1S, and this was the perfect first projec to christen it. Hopefully that next one isn't too far away!!

My Github [repo is here with all the code and hardware - when it comes out!](https://github.com/TheOracle2212/VFD-Clock)
