There's a  **Python App**  in each edge board (Jetson Nano), the App takes charge to analyze camera live video stream, and upload analyzed message to a _kafka server_  periodically with the detected objects info.

# Upload criteria 

Camera live video stream is real-time send into edge board for detection analyze, only when pre-defined(door sign, people, electric bicycle) objects are detected will trigger uploading. 

# Upload interval

for performance consideration, uploading is not real-time but by a interval, the interval now is around  _2 seconds_ , and could be changed in future. 

> current camera live video stream is 24FPS, and the App is uploading(if objects detected) at every 48 frame, the objects in other frames will be ignored. 

# Kafka server info

Url:
> dev-iot.ipos.biz

> port: 9092

Topic: 
> test

# Message samples

- ONLY 2 door sign were detected

this is the most common case that means the elevator door are closed, and no one in it, indicates the elevator is in `idle` or `on calling`.

> idle: the elevator is not moving, and door closed.

> on calling: the elevator is going to floor because someone pressed  _go down_  or  _up_ button to ask a lift.

there're 2 moving doors for  _most_  (but 1 is possible) elevators, so the `door sign` count is 2 here.

> NOTE, the `Vehicle` string value here is dummy, please ignore.
```

{'version': '4.0', 'id': 41088, 
'@timestamp': '2022-02-07T05:01:06.354Z', 
'sensorId': 'shaoLocal2GJtsBoard', 
'objects': [
    '18446744073709551615|383.927|69.3955|449.859|119.981|Vehicle|#|DoorWarningSign|B|M|y|l|CN|0.460956', #detect door sign one, NOTE, the `Vehicle` here is dummy
    '18446744073709551615|306.012|76.0776|376.832|130.611|Vehicle|#|DoorWarningSign|B|M|y|l|CN|0.622918'] #detect door sign two, NOTE, the `Vehicle` here is dummy
}


```

- 1 electric bicycle + 3 person + no door sign were detected

no door sign indicates the door is in opened state (could not see the door sign at the camera).

```

{'version': '4.0', 'id': 20112, 
'@timestamp': '2022-02-07T05:01:05.134Z',                   #the time the detection occurred
'sensorId': 'default_shaoLocalJsNxBoard',                   #the unique id binding to a board, this is set in board local config.
'objects': [
    '18446744073709551615|279.561|197.867|476.846|503.844|Vehicle|#|TwoWheeler|B|M|b|X|CN|0.981058',   #detect a two wheeler, aka electric bicycle                      
    '18446744073709551615|354.695|42.6953|669.148|459.346|Person|#|m|18|b|n|f|0.248717',               #detect person one
    '18446744073709551615|434.297|151.125|597.471|483.01|Person|#|m|18|b|n|f|0.999874',                #detect person two
    '18446744073709551615|365.938|117.453|575.322|429.719|Person|#|m|18|b|n|f|0.999982']               #detect person three
}

```

- 1 person + 2 door sign were detected

door sign indicates the door is in closed state and carried a person, this is the case that elevator is about to moving to target floor.
```

{'version': '4.0', 'id': 19104, '@timestamp': '2022-02-07T05:00:24.822Z', 
'sensorId': 'default_shaoLocalJsNxBoard', 
'objects': ['18446744073709551615|5.87891|333.82|512.148|822.59|Person|#|m|18|b|n|f|1', 
    '18446744073709551615|147.618|97.7041|226.539|154.511|Vehicle|#|DoorWarningSign|B|M|y|l|CN|0.570212', 
    '18446744073709551615|517.02|61.1372|584.524|105.502|Vehicle|#|DoorWarningSign|B|M|y|l|CN|0.687945']}

```