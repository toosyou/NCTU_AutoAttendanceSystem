# NCTU AutoAttendanceSystem

1. Login to [NCTU Attendance System](https://pt-attendance.nctu.edu.tw/)
1. Check your remain time in hours.
1. Press F12 to open the chrome console.
1. Click into 「填寫 / 修改 (工作/服務) 日誌」.
1. Find your right project id by observing the html in the console. It might be something like  
`<option value="108TXXX^21XXXX^^96XXX"> 108TXXX: 2020-XX-XX~2020-XX-XX -  請核單號:XXXXX </option>`,
  then the project id you need is ```108TXXX^21XXXX^^96XXX```.
1. Modify the following script and copy it to the console.

```javascript
window.confirm = () => true;
(async () => {
    function sleep(milliseconds){
        return new Promise((resolve, reject) => {
            setTimeout(() => resolve(), milliseconds);
        })
    }
    function generate_start_and_end_time(month, date, is_morning, time){
        var start_time = "";
        var end_time = "";
        var date_string = "2020-" + month + "-" + String(date).padStart(2, "0") + " ";
        if(is_morning){
            start_time =  date_string + "08:00:00";
            end_time = date_string + String(8 + Math.floor(time)).padStart(2, "0") + ":" +
                                    ((time % 1 == 0) ? "0" : "30") + ":00";
        }
        else{
            start_time =  date_string + "17:00:00";
            end_time = date_string + String(17 + Math.floor(time)).padStart(2, "0") + ":" +
                                    ((time % 1 == 0) ? "0" : "30") + ":00";
        }

        return [start_time, end_time];
    }

    var month = "04";
    var date = 2; // first day of the month
    var weekday = 4; // first day of the month
    var time_remain = 71.5; // your remain time in hours
    var project_name = "INSERT PROJECT ID";
    var jobs_name = "課程助教";
    var use_morning = false; // if the time_remain is too large, turn this on to use the morning.
    var is_morning = use_morning;
    var TIME_CHOISES = [4, 3, 2, 1, 0.5];

    while(time_remain > 0){
        // calculate time
        var start_time = "";
        var end_time = "";

        if(weekday >= 1 && weekday <= 5){
            for(var i = 0; i < TIME_CHOISES.length; i++){
                if(time_remain >= TIME_CHOISES[i]){
                    const [tmp_start, tmp_end] = generate_start_and_end_time(
                                                    month, date, is_morning, TIME_CHOISES[i]);
                    start_time = tmp_start;
                    end_time = tmp_end;
                    time_remain -= TIME_CHOISES[i];
                    break;
                }
            }
            console.log(time_remain);
            console.log(start_time);
            console.log(end_time);

            document.querySelector("#pno").value = project_name;
            document.querySelector("#datetimepicker1").value = start_time;
            document.querySelector("#datetimepicker2").value = end_time;
            document.querySelector("#workdata4").value = jobs_name;
            document.querySelector("#btnSubmit").click();
            await sleep(2000);
        }
        if(use_morning && is_morning){
            is_morning = false;
        }
        else{
            date += 1;
            weekday = (weekday % 7) + 1;
            is_morning = use_morning;
        }
    }
    document.querySelector("#node_level-1-2").click(); // switch
    await sleep(2000);
    document.querySelector("#grid_grid_check_all").click();
    await sleep(2000);
    document.querySelector("#btnSubmit").click();
})()
```
