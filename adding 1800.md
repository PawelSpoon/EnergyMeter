diese message geht an die db 
{
    "hms-1800_total_P_AC":1162.9,
    "hms-1800_total_YieldTotal":1351.847,
    "hms-1800_total_YieldDay":6305,
    "hms-1800_total_is_valid":1
}

das ist die message vom 1500er
{
    "hm-1500_total_U_AC":241.3,
    "hm-1500_total_I_AC":2.94,
    "hm-1500_total_P_AC":708.1,
    "hm-1500_total_Q_AC":23.3,
    "hm-1500_total_F_AC":49.99,
    "hm-1500_total_PF_AC":0.999,
    "hm-1500_total_Temp":43.9,
    "hm-1500_total_ALARM_MES_ID":365,
    "hm-1500_total_YieldDay":1012,
    "hm-1500_total_YieldTotal":877.308,
    "hm-1500_total_P_DC":745.3
}

// org
if (msg.topic.split("/")[1] === "ac") {
    msg.key = "hms-1800_" + msg.topic.split("/")[2]
    msg.payload = Number(msg.payload)
    return msg;
}
return msg;

// new trial
if (msg.topic.split("/")[1] === "ac") {
    if (msg.topic.split("/")[2] === "power") {
        msg.key = "hms-1800_total_P_AC"
    } else if (msg.topic.split("/")[2] === "yieldtotal") {
        msg.key = "hms-1800_total_YieldTotal"
    } else if (msg.topic.split("/")[2] === "yieldday") {
        msg.key = "hms-1800_total_YieldDay"
    } else {
        msg.key = "hms-1800_total_" + msg.topic.split("/")[2]
    }
    msg.payload = Number(msg.payload)
    return msg;
}
return msg;



if (msg.topic.split("/")[1] === "hm-1500") {
    if (msg.topic.split("/")[2] === "ch0") {
    msg.key = msg.topic.split("/")[1] + "_total_" + msg.topic.split("/")[3]
    msg.payload = Number(msg.payload)
    return msg;
    }
}
return msg;