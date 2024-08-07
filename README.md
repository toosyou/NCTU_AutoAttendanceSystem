# NCTU AutoAttendanceSystem

Instructions for Automatically Generating Attendance Records in NCTU Attendance System:

1. Log in to the [NCTU Attendance System](https://pt-attendance.nctu.edu.tw/).
2. Open the Chrome Console by pressing `F12`.
3. Copy the script provided below and paste it into the console. Then, press Enter.
4. Select the correct project from the dropdown list and press "2. 自動產生" to generate the attendance records.
5. Check the generated records carefully and submit them under the "產生（簽到 / 服務）資料" tag.
* Note: While I have not encountered any issues using this method, please be aware that you are using it at your own risk.

```javascript
(async () => {
    function sleep(milliseconds){
        return new Promise((resolve, reject) => {
            setTimeout(() => resolve(), milliseconds);
        })
    }

    var project_name, project_index;
    document.querySelector("#node_level-1-1").onclick();
    $('#showWorkLists').modal("hide")
    await sleep(2000);

    window.confirm = () => true;
    document.querySelector("#pno").onchange = () => {
        showBugetName();
        project_index = document.querySelector("#pno").selectedIndex;
        project_name = document.querySelector("#pno").value;
        document.querySelector("#payType").style.color = "black";
    };

    document.querySelector("#btnClear").value = "2. 自動產生";
    document.querySelector("#btnClear").style.color = "red";
    document.querySelector("#btnClear").onclick = async () => {
        console.log("Here we go!");
        var date_string = document.querySelector("#pno").options[project_index].text;
        var start_date = new Date(date_string.split(" ")[1].split("~")[0]);
        var end_date = new Date(date_string.split("~")[1].split(" ")[0]);
        var work_content = $('#workdata4').val();

        generate_records(project_name, start_date, end_date, work_content);
    }
    
    function get_worklists(){
        return $.ajax({
            async: false,
            url: 'ajaxfunction.php',
            cache: false,
            dataType: 'json',
            type: 'POST',
            async: !1,
            data:
            {
                actionFunction: 'getWorkLists'
            }
        }).responseJSON;
     }

    function get_total_time(project_name){
        worklists = get_worklists();
        
        bugetno = project_name.split("^")[0];
        SerialNo = project_name.split("^")[3];

        times = [];

        for(var i=0; i<worklists.length; ++i){
            if(worklists[i]["bugetno"] == bugetno && worklists[i]["SerialNo"] == SerialNo)
                times.push(parseInt(worklists[i]["LimitPerMonth"]));
        }
        return Math.max(...times);
    }

    async function generate_records(project_name, start_date, end_date, work_content){
        var time_remain = get_total_time(project_name) + 0.5;
        var total_time = time_remain;
        var now = new Date(start_date.getTime()); // copy
        while(time_remain > 0){
            var year = now.getFullYear();
            var month = now.getMonth() + 1;
            var date = now.getDate();
            var work_time_sofar = 0;
        
            var date_string = year + "-" + String(month).padStart(2, "0") + "-" + String(date).padStart(2, "0") + " ";
            var start_time = "", end_time ="";

            for(var hour=8; hour <= 21 && time_remain > 0 && work_time_sofar < 8; ++hour){
                if(hour == 12) continue; // launch break

                var minute = hour < 17 ? 10 * Math.floor((hour - 8 - (hour > 12)) / 2) : 30;
                
                start_time = date_string + String(hour).padStart(2, "0") + ":" + String(minute).padStart(2, "0") + ":00";
                if(time_remain >= 1)
                    end_time = date_string + String(hour+1).padStart(2, "0") + ":" + String(minute).padStart(2, "0") + ":00";
                else{
                    var end_hour = hour + (minute + 30 >=60);
                    var end_minute = (minute + 30) % 60;
                    end_time = date_string + String(end_hour).padStart(2, "0") + ":" + String(end_minute).padStart(2, "0") + ":00";
                }

                // try to fill int the time section
                document.querySelector("#pno").value = project_name;
                document.querySelector("#datetimepicker1").value = start_time;
                document.querySelector("#datetimepicker2").value = end_time;
                $('#workdata4').val(work_content);

                // perform checks
                if(CheckClass() == "N") continue; // there are classes in the time section
                if(check5Days() != "Y") break; // work more than 5 days this week
                if(checkOverWorkTime() != "Y"){time_remain = 0; break;} // finished

                addRecord(); // submit
                await sleep(00);

                // progress bar
                var progress = parseInt((total_time - time_remain) / total_time * 100);
                var n_dots = Math.floor(progress / 5);
                const dots = ".".repeat(n_dots);
                const empty = " ".repeat(20 - n_dots);
                console.log(`\r[${dots}${empty}] ${progress}% (remain: ${time_remain} h)`);

                time_remain -= 1;
                work_time_sofar += 1;
            }
            now.setDate(now.getDate() + 1); // the next day
        }
        console.log("Finished!");
    }

    console.log("Please select a project and press 自動產生!");
    document.querySelector("#payType").textContent = "<------- 1. 選擇計畫";
    document.querySelector("#payType").style.color = "red";

    // remove distraction
    document.querySelector("#form1 > font:nth-child(27)").style.color = "black";
    document.querySelector("#form1 > font:nth-child(29)").style.color = "black";
    document.querySelector("#form1 > font:nth-child(31)").style.color = "black";
})()
```
