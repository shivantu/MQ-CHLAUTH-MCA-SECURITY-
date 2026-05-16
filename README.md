# 🔐 IBM MQ 9.4 — AUTHINFO + CHLAUTH + MCAUSER QUICK REFERENCE

---

# 📘 SCENARIO

| Component | Value |
|---|---|
| Queue Manager | QM1 |
| Channel | APP.SVRCONN |
| Queue | TEST.Q1 |
| Application Users | appusr1, appusr2 |
| Technical User | mqapp |

---

# 🏗️ CORE SECURITY FLOW

```text
Application User
      |
      v
AUTHINFO
(verify password)
      |
      v
CHLAUTH
(allow/map user)
      |
      v
MCAUSER
(MQ runtime identity)
      |
      v
OAM
(permission check)
      |
      v
Queue Access

---

🚀 STEP 1 — CREATE OS USERS

Create Application Users

useradd appusr1
useradd appusr2

Set Passwords

passwd appusr1
passwd appusr2

Purpose

Users used for MQ client authentication.

---

Create Technical MCAUSER

useradd mqapp

Purpose

MQ internal runtime authorization user.

---

🚀 STEP 2 — CREATE QUEUE MANAGER

Create QM

crtmqm QM1

Start QM

strmqm QM1

Purpose

Creates and starts Queue Manager.

---

🚀 STEP 3 — CREATE QUEUE

runmqsc QM1

DEFINE QLOCAL(TEST.Q1)

Purpose

Application target queue.

---

🚀 STEP 4 — CREATE SVRCONN CHANNEL

DEFINE CHANNEL(APP.SVRCONN) CHLTYPE(SVRCONN) TRPTYPE(TCP)

Purpose

Allows remote client applications to connect.

---

🚀 STEP 5 — CREATE LISTENER

DEFINE LISTENER(TCP.LSTR) TRPTYPE(TCP) PORT(1414) CONTROL(QMGR)

START LISTENER(TCP.LSTR)

Purpose

Accepts client TCP connections.

---

🚀 STEP 6 — CREATE AUTHINFO

DEFINE AUTHINFO(APP.AUTHINFO) AUTHTYPE(IDPWOS) CHCKCLNT(REQUIRED)

Meaning

Parameter| Meaning
AUTHTYPE(IDPWOS)| Verify OS username/password
CHCKCLNT(REQUIRED)| Password mandatory

Purpose

Authenticates client username/password.

---

🚀 STEP 7 — ENABLE AUTHENTICATION

ALTER QMGR CONNAUTH(APP.AUTHINFO)

REFRESH SECURITY TYPE(CONNAUTH)

Purpose

Activates connection authentication.

---

🚀 STEP 8 — CHLAUTH USER MAPPING

SET CHLAUTH(APP.SVRCONN) TYPE(USERMAP) CLNTUSER('appusr1') USERSRC(MAP) MCAUSER('mqapp')

SET CHLAUTH(APP.SVRCONN) TYPE(USERMAP) CLNTUSER('appusr2') USERSRC(MAP) MCAUSER('mqapp')

Meaning

Application users are mapped to mqapp.

Purpose

Centralized MQ authorization using one technical user.

---

🚀 STEP 9 — REMOVE FIXED CHANNEL MCAUSER

ALTER CHANNEL(APP.SVRCONN) MCAUSER('')

Purpose

Allows CHLAUTH dynamic mapping to work properly.

«MCAUSER('') means:
No fixed user at channel level.»

---

🚀 STEP 10 — GIVE QM AUTHORITY

setmqaut -m QM1 -t qmgr -p mqapp +connect +inq

Purpose

Allows mqapp to connect to Queue Manager.

---

🚀 STEP 11 — GIVE QUEUE AUTHORITY

setmqaut -m QM1 -n TEST.Q1 -t queue -p mqapp +put +get +browse +inq

Purpose

Allows queue operations.

---

🔌 APPLICATION CONNECTION DETAILS

Property| Value
Host| MQ_SERVER
Port| 1414
Channel| APP.SVRCONN
Username| appusr1
Password| Test@123
Queue Manager| QM1

---

⚙️ INTERNAL MQ PROCESSING

1. Application sends username/password
2. AUTHINFO verifies password
3. CHLAUTH checks mapping rule
4. MQ maps user → mqapp
5. OAM checks mqapp permissions
6. Queue access granted

---

📚 IMPORTANT UNDERSTANDING

AUTHINFO

Purpose

Password verification.

Checks

«"Who are you?"»

---

CHLAUTH

Purpose

Allow/block/map users.

Checks

«"Can you use this channel?"»

---

MCAUSER

Purpose

MQ runtime authorization identity.

Checks

«"Which MQ user should run this connection?"»

---

OAM

Purpose

Checks MQ permissions.

Checks

«"What operations are allowed?"»

---

🔐 FIXED CHANNEL MCAUSER EXAMPLE

ALTER CHANNEL(APP.SVRCONN) MCAUSER('mqapp')

Meaning

ALL users become mqapp internally.

Example

user1 -> mqapp
user2 -> mqapp
user3 -> mqapp

---

✅ BEST PRACTICE

1. Use dedicated technical MCAUSER
2. Never use mqm as MCAUSER
3. Enable AUTHINFO
4. Use CHLAUTH mapping
5. Use TLS in production
6. Use least privilege access
7. Monitor 2035 authorization failures

---

🛠️ IMPORTANT DISPLAY COMMANDS

Show AUTHINFO

echo "DISPLAY AUTHINFO(*)" | runmqsc QM1

Show CONNAUTH

echo "DISPLAY QMGR CONNAUTH" | runmqsc QM1

Show CHLAUTH

echo "DISPLAY CHLAUTH(APP.SVRCONN)" | runmqsc QM1

Show Channel MCAUSER

echo "DISPLAY CHANNEL(APP.SVRCONN) MCAUSER" | runmqsc QM1

Show Queue Authorities

dspmqaut -m QM1 -t queue -n TEST.Q1

Show Queue Manager Authorities

dspmqaut -m QM1 -t qmgr

Show Listener Status

echo "DISPLAY LSSTATUS(TCP.LSTR)" | runmqsc QM1

Show Channel Status

echo "DISPLAY CHSTATUS(APP.SVRCONN)" | runmqsc QM1

---

⚠️ COMMON ERRORS

2035 MQRC_NOT_AUTHORIZED

Cause

- Wrong password
- Missing authority
- Blocked by CHLAUTH

Fix

Verify:

- AUTHINFO
- CHLAUTH
- setmqaut permissions

---

AMQ5534 USER NOT AUTHORIZED

Cause

No Queue Manager connect authority.

Fix

setmqaut -m QM1 -t qmgr -p mqapp +connect

---

2538 HOST NOT AVAILABLE

Cause

Listener not running.

Fix

START LISTENER(TCP.LSTR)

---
