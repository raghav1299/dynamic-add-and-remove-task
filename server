#!/usr/bin/python3
from flask import Flask, request, jsonify
from flask_cors import CORS
import boto3
import sys
import ssl
import json
import logging, traceback
import pymysql
import datetime
import sched, time
import threading
import time
app = Flask(__name__)
cors=CORS(app,resources={r'/*':{'origins':'*'}})
cog = boto3.client('cognito-idp', region_name='ap-south-1')
cogcli='2u1lch909cct1h3sftf1k9te2a'
@app.route('/signup',methods=['GET', 'POST'])
def signup():
    try:
        print('Entered')
        content = json.loads(request.data)
        email = str(content['email'])
        x=0
        cogcli='2u1lch909cct1h3sftf1k9te2a'
        if(email.rfind("srmist.edu.in")!=-1 or email.rfind("srmuniv.edu.in")!=-1):
            cog.sign_up(
                ClientId = cogcli,
                Username = content['username'],
                Password = content['password'],
                UserAttributes=[{'Name':'email','Value':content['email']}]
            )
            x=1
    except cog.exceptions.UsernameExistsException:
        return 'Username Already Exists!'
    except Exception as e:
        print('Exception: ' +str(e))
        return str(e)
    if(x==1):
        return 'True'
    else:return 'False'

@app.route('/login', methods=['GET', 'POST'])
def login():
    try:
        print(request.data)
        content = json.loads(request.data)
        print(content)

        auth = cog.initiate_auth(
                AuthFlow = 'USER_PASSWORD_AUTH',
                AuthParameters = {
                        'USERNAME': content['username'],
                        'PASSWORD': content['password']
                },
                        ClientId = cogcli
        )
        return json.dumps(auth['AuthenticationResult'])
    except cog.exceptions.UserNotConfirmedException:
        return 'User is not confirmed. Please check your mail.'
    except cog.exceptions.UserNotFoundException:
        return 'User does not exist. Check again.'
    except cog.exceptions.NotAuthorizedException:
        return 'Username/Password is incorrect'
    except Exception as e:
        print(e)
        return str(e)

def dtSerializer(obj):
     if isinstance(obj, datetime.datetime):
             return(obj.isoformat())
     else:
             TypeError('Unknown serializer')

@app.route('/logout', methods=["GET", "POST"])
def logout():
    try:
        content = json.loads(request.data)
        cog.global_sign_out(AccessToken=content['AccessToken'])
        return "true"
    except Exception as e:
        print(e)
        return "true"

@app.route('/forgotpassword', methods=["GET", "POST"])
def forgotpassword():
    try:
        content = json.loads(request.data)
        cog.forgot_password(ClientId=cogcli, Username=content['Username'])
    except cog.exceptions.UserNotFoundException:
        return "User does not exist"
    except Exception as e:
        print(e)
        return "False"
    return "True"

@app.route('/confirmPass', methods=["GET", "POST"])
def confirmPass():
    try:
        content = json.loads(request.data)
        cog.confirm_forgot_password(ClientId=cogcli, Username=content['Username'], ConfirmationCode=content['code'], Password=content['Password'])
    except cog.exceptions.UserNotFoundException:
        return "User does not exist. Try Again!"
    except cog.exceptions.InvalidParameterException:
        return "Email not verified"
    except cog.exceptions.CodeMismatchException:
        return "Invalid Confirmation Code"
    except Exception as e:
        print(e)
        return "False"
    return "True"

@app.route('/ChangePassword', methods=["GET", "POST"])
def ChangePassword():
def ChangePassword():
    try:
        content = json.loads(request.data)
        cog.change_password( PreviousPassword=content['Password'],ProposedPassword=content['NewPassword'],AccessToken=content['AccessToken'])
    except cog.exceptions.UserNotFoundException:
        return "User does not exist"
    except cog.exceptions.InvalidParameterException:
        return "Email not verified"
    except cog.exceptions.InvalidPasswordException:
        return "Invalid Password"
    except Exception as e:
        print(e)
        return "False"
    return "True"

@app.route('/getDetails', methods=['GET', 'POST'])
def getDetails():
        content = json.loads(request.data)
        conn = pymysql.connect(
        host='water-db.c8iptqv0aosa.ap-south-1.rds.amazonaws.com',
                user='admin',
        password='admin123',
        database='water'
        )
        conn.autocommit(True)
        total=[]
        with conn.cursor(pymysql.cursors.DictCursor) as cur:
                cur.execute('SELECT Floor,water.tap_id,sum(Total) total from water.water JOIN water.water1 on(water.tap_id=water1.tap_id) where Floor= "'+ content['Flooor']+ '" AND Building= "'+  content['Building']+ '" AND tnd >="' + str(content['from']) + '" AND tnd<="' + str(content['to']) + '" group by tap_id')
                if cur.rowcount!=0:
                        total= cur.fetchall()
                cur.execute('select tap_id from water.water1 where Floor= "'+ content['Floor']+ '" AND Building= "'+  content['Building']+ '"')
                tap=cur.fetchall()
        p1={"Total":total}
        p={"Total":tap}
        for i in range(len(p["Total"])):
            c=0
            ele1=p["Total"][i]["tap_id"]
            for j in range(len(p1["Total"])):
                ele2=p1["Total"][j]["tap_id"]
 if(ele1==ele2):
                    c=c+1
            if(c==0):
                p1["Total"].append({"Floor":content['Floor'], "tap_id":ele1, "total":0.0})
        conn.close()
        return json.dumps(p1,default=dtSerializer)
@app.route('/buildpage',methods=['GET','POST'])
def buildpage():
        content=json.loads(request.data)
        conn = pymysql.connect(
        host='water-db.c8iptqv0aosa.ap-south-1.rds.amazonaws.com',
        user='admin',
        password='admin123',
        database='water'
        )
        conn.autocommit(True)
        total=[]
        with conn.cursor(pymysql.cursors.DictCursor) as cur:
                cur.execute('select Building,sum(Total) total from water.water JOIN water.water1 on(water.tap_id=water1.tap_id) where tnd>="'+str(content['from'])+'" AAND tnd<="'+str(content['to'])+'"group by Building')
                if cur.rowcount!=0:
                        total=cur.fetchall()
                cur.execute('select Building from water.water1')
                build=cur.fetchall()
        p1={"Total":total}
        p={"Total":build}
        for i in range(len(p["Total"])):
            c=0
            ele1=p["Total"][i]["Building"]
            for j in range(len(p1["Total"])):
                ele2=p1["Total"][j]["Building"]
                if(ele1==ele2):
                    c=c+1
            if(c==0):
                p1["Total"].append({"Building":ele1, "total":0.0})
        payload={
                 'Total':total}
        conn.close()
        return json.dumps(p1,default=dtSerializer)
@app.route('/secpage',methods=['GET','POST'])
def secpage():
        content=json.loads(request.data)
        conn = pymysql.connect(
        host='water-db.c8iptqv0aosa.ap-south-1.rds.amazonaws.com',
        user='admin',
        password='admin123',
        database='water'
        )
        conn.autocommit(True)
        total=[]
        with conn.cursor(pymysql.cursors.DictCursor) as cur:
                cur.execute('select Building,Floor,sum(Total) total from water.water JOIN water.water1 on(water.tap_id=water1.tap_id) where tnd>="'+str(content['from'])+'" AND tnd<="'+str(content['to'])+'"AND Building="'+content['Building']+'" group by Building,Floor')
                if cur.rowcount!=0:
                        total=cur.fetchall()
                cur.execute('select Floor from water.water1 where Building="'+  content['Building']+ '"')
                floor=cur.fetchall()
        p1={"Total":total}
        p={"Total":floor}
        for i in range(len(p["Total"])):
            c=0
            ele1=p["Total"][i]["Floor"]
            for j in range(len(p1["Total"])):
                ele2=p1["Total"][j]["Floor"]
                if(ele1==ele2):
                    c=c+1
            if(c==0):
                p1["Total"].append({"Building":content['Building'], "Floor":ele1, "total":0.0})
        conn.close()
        return json.dumps(p1,default=dtSerializer)
@app.route('/dropbuild',methods=['GET','POST'])
def dropbuild():
        conn = pymysql.connect(
        host='water-db.c8iptqv0aosa.ap-south-1.rds.amazonaws.com',
        user='admin',
        password='admin123',
        database='water'
        )
        conn.autocommit(True)
        with conn.cursor(pymysql.cursors.DictCursor) as cur:
                cur.execute('select Floor from water.water1 where Building="'+str(content['Building'])+'"')
                x=cur.fetchall()
        payload={'Floor':x}
        conn.close()
        return json.dumps(payload,default=dtSerializer)
@app.route('/complaint',methods=['GET','POST'])
def complaint():
        content = json.loads(request.data)
        conn = pymysql.connect(
        host='water-db.c8iptqv0aosa.ap-south-1.rds.amazonaws.com',
        user='admin',
        password='admin123',
        database='water'
        )
        x={"Complaints":""}
        with conn.cursor(pymysql.cursors.DictCursor) as cur:
                cur.execute('select * from water.water2')
                if cur.rowcount!=0:
                        x=cur.fetchall()
        payload = {'Complaints':x}
        conn.close()
        return json.dumps(payload,default=dtSerializer)

@app.route('/inscomplaint',methods=['GET','POST'])
def inscomplaint():
        content = json.loads(request.data)
        conn = pymysql.connect(
        host='water-db.c8iptqv0aosa.ap-south-1.rds.amazonaws.com',
        user='admin',
        password='admin123',
        database='water'
        )
        conn.autocommit(True)
        with conn.cursor(pymysql.cursors.DictCursor) as cur:
                cur.execute('insert into water.water2(Building,Floor,location,Complaint) values ("'+str(content['Building'])+'","'+str(content['Floor'])+'","'+str(content['location'])+'","'+str(content['Complaint'])+'")')
        payload = {'Complaints':'asd'}
        conn.close()
        return json.dumps(payload,default=dtSerializer)

@app.route('/delcomplaint',methods=['GET','POST'])
def delcomplaint():
        content = json.loads(request.data)
        conn = pymysql.connect(
        host='water-db.c8iptqv0aosa.ap-south-1.rds.amazonaws.com',
        user='admin',
        password='admin123',
        database='water'
        )
        conn.autocommit(True)
        if(content['username']=="management"):
                with conn.cursor(pymysql.cursors.DictCursor) as cur:
                        cur.execute('delete from water.water2 where Building="'+str(content['Building'])+'" and Floor= "'+str(content['Floor'])+'" and location="'+str($
                payload={'Complaints':'1'}
                return json.dumps(payload,default=dtSerializer)
        else:
                payload={'Complaints':'0'}
                return json.dumps(payload,default=dtSerializer)
        conn.close()
@app.route('/filter',methods=['GET','POST'])
def filter():
        conn = pymysql.connect(
        host='water-db.c8iptqv0aosa.ap-south-1.rds.amazonaws.com',
        user='admin',
        password='admin123',
        database='water'
        )
        conn.autocommit(True)
        content=json.loads(request.data)
        with conn.cursor(pymysql.cursors.DictCursor) as cur:
                cur.execute('select Building,Floor,location,Complaint from water.water2 where Building="'+str(content['Building'])+'"')
                x=cur.fetchall()
        payload = {'filter':x}
        conn.close()
        return json.dumps(payload,default=dtSerializer)
@app.route('/building',methods=['GET','POST'])
def building():
        conn = pymysql.connect(
        host='water-db.c8iptqv0aosa.ap-south-1.rds.amazonaws.com',
        user='admin',
        password='admin123',
        database='water'
        )
        conn.autocommit(True)
        content = json.loads(request.data)
        x={"Building":""}
        with conn.cursor(pymysql.cursors.DictCursor) as cur:
                cur.execute('select Building from water.water2 group by Building')
                if cur.rowcount!=0:
                        x=cur.fetchall()
        payload = {'Building':x}
        conn.close()
        return json.dumps(payload,default=dtSerializer)
@app.route('/showDelete',methods=['GET','POST'])
def showDelete():
        content=json.loads(request.data)
        if(content['username']=="management"):
                payload={"Show":1}
        else:
                payload={"Show":0}
        return json.dumps(payload,default=dtSerializer)

if __name__ == '__main__':
        app.run(host='0.0.0.0', debug=True)
