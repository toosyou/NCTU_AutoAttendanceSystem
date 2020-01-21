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

    var month = "04";
    var date = 2; // first day of the month
    var weekday = 2; // first day of the month
    var time_remain = 37.5; // your remain time in hours
    var project_name = "INSERT PROJECT ID";
    var jobs_name = "課程助教";

    while(time_remain > 0){
        // calculate time
        var start_time = "";
        var end_time = "";

        if(weekday >= 1 && weekday <= 5){
            if(time_remain >= 4){
                start_time = "2020-" + month + "-" + String(date).padStart(2, "0") + " " +  "17:30:00";
                end_time = "2020-" + month + "-" + String(date).padStart(2, "0") + " " +  "21:30:00";
                time_remain -= 4;
            }else if(time_remain >= 1.5) {
                start_time = "2020-" + month + "-" + String(date).padStart(2, "0") + " " +  "17:30:00";
                end_time = "2020-" + month + "-" + String(date).padStart(2, "0") + " " +  "19:00:00";
                time_remain -= 1.5;
            }else if(time_remain >= 1){
                start_time = "2020-" + month + "-" + String(date).padStart(2, "0") + " " +  "17:30:00";
                end_time = "2020-" + month + "-" + String(date).padStart(2, "0") + " " +  "18:30:00";
                time_remain -= 1;
            }else if(time_remain >= 0.5){
                start_time = "2020-" + month + "-" + String(date).padStart(2, "0") + " " +  "17:30:00";
                end_time = "2020-" + month + "-" + String(date).padStart(2, "0") + " " +  "18:00:00";
                time_remain -= 0.5;
            }
            console.log(time_remain)

            document.querySelector("#pno").value = project_name;
            document.querySelector("#datetimepicker1").value = start_time;
            document.querySelector("#datetimepicker2").value = end_time;
            document.querySelector("#workdata4").value = jobs_name;
            document.querySelector("#btnSubmit").click();
            await sleep(2000);
        }

        date += 1;
        weekday = (weekday % 7) + 1;
    }
    document.querySelector("#node_level-1-2").click(); // switch
    await sleep(2000);
    document.querySelector("#grid_grid_check_all").click();
    await sleep(2000);
    document.querySelector("#btnSubmit").click();
})()
```
