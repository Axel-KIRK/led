#!/bin/bash

POWERON='{"method": "set_power", "param": {"value": true}}'
POWEROFF='{"method": "set_power", "param": {"value": false}}'
GETSTATUS='{"method": "get_status"}'

calc(){ awk "BEGIN { print int(${1}*100/${2}}/100 }";}

function send() {
    local JSON="${1}"
    r=$(curl -X POST http://192.168.1.80/command \
    --connect-timeout 1 \
    --max-time 1 \
    --write-out "%{http_code}\n" -s -o /dev/null \
    -H 'Content-Type: application/json'
    -d "$JSON")
    echo $r
}


function post() {
    local JSON="${1}"
    result=$(send "$JSON");
    echo "$result"
    if [[ "$result" -ne "200" ]]; then
        try=$(send "$POWERON")
        while [[ "$try" -ne "200" ]]; do
            echo "try restart"
            sleep 5
            try=$(send "$POWERON")
        done
    fi
}

function setColor(){
    local r=$(calc ${1} 255)
    echo "r $r"
    local g=$(calc ${2} 255)
    local b=$(calc ${3} 255)
    local w=$(calc ${4} 255)
    post "{'method': 'set_rgbw','param': {'r': "$r",'g':"$g",'b': "$b",'w': "$w", time:0}}"
}

post "$POWEROFF"
sleep 1
setColor 0 0 0 0
sleep 1
post "$POWERON"
for i in {1..50}
do
    setColor $i 0 0 0
    sleep 5
done
for i in {1..20}
do
    setColor 50 $i 0 0
    sleep 5
done
sleep 600
post "$POWEROFF"
