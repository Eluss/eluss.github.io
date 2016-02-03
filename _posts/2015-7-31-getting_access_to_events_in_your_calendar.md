---
layout: post
title: EventKit - Gaining access to calendar events
---
If you want to gain access to your calendar events it's time to use EventKit. EventKit allows you to do multiple operations with calendar events and reminders. In this post I will cover gaining access to existing events in your calendar.

Getting access to events in your calendar
---
In order to get desired access you need to follow a few basic steps.

1. Your app needs to gain access to calendar. This code will show the alert to a user when app is opened for the 	first time. You can always change it in app's settings later on.
	<script src="https://gist.github.com/Eluss/ce81bc07f90cb7c14055.js"></script>

2. Getting a list of calendars available on your device. Using this method you will get an array of 'EKCalendar' 	type objects.
	<script src="https://gist.github.com/Eluss/1533b5aa461ddc612cab.js"></script>

3. Getting events from your calendars. This way you get array of EKEvent objects describing events in	 your calendar.
	<script src="https://gist.github.com/Eluss/1a788b0d3d3d310ca1cf.js"></script>
