Sawtooth Channel Example
========================

This is an example of a Hyperledger Sawtooth blockchain network,
for the purpose of demonstrating the Sawtooth
Generic Discrete Message Transaction Processor
(https://github.com/trustbridge/sawtooth-channel-dgm-tp).

Together, this provides a G2G channel component as described at http://edi3.org 

The code in this repository draws heavily on Apache-2 licenced examples
from https://github.com/hyperledger


Running local demo
------------------

This assumes you have a working docker environment and docker-compose.

**1. start up services with docker-compose.**

sawtooth-channel-dgm-tp is inculded as a submodule,
so ensure your copy exists and is up to date:
::
  git submodule update

Then run it with docker compose:
::
  docker-compose -f sawtooth-channel-dgm-tp/local.yml up --build --force

This runs up the following services:

* ``validator`` 
* ``rest-api``
* ``identity transaction processor``
* ``settings transaction processor``
* ``generic discrete message transaction processor``
* ``client``

**2. start up bash session on client container**

:: 
  
  docker exec -it sawtooth-shell-local bash
  
**3. create a transaction from within client container.**

``gdm send`` expects sender_ref, subject, object, predicate, sender, receiver in that order. 

:: 

  python3 bin/gdm send 123456789 somechamber create skljhdfkjhskdjhksjhf AU CN --url http://rest-api:8008

You should get a response like this:: 

  Response: {
    "link": "http://rest-api:8008/batch_statuses?id=fc9d35c7ae17191f02705a166cee84475f0d2d462e9e1e9cf637debbb3aca17c2a8c44d08c4c18ef51f6a4a4b447ce26d9d4f4f9e4df32155e671d8d744fba8d"
  }


**5. you can then use the link to get the status of the batch**

:: 

  curl http://rest-api:8008/batch_statuses?id=fc9d35c7ae17191f02705a166cee84475f0d2d462e9e1e9cf637debbb3aca17c2a8c44d08c4c18ef51f6a4a4b447ce26d9d4f4f9e4df32155e671d8d744fba8d

the response should tell you whether the batch has been added to the blockchain

:: 

  {
    "data": [
      {
        "id": "fc9d35c7ae17191f02705a166cee84475f0d2d462e9e1e9cf637debbb3aca17c2a8c44d08c4c18ef51f6a4a4b447ce26d9d4f4f9e4df32155e671d8d744fba8d",
        "invalid_transactions": [],
        "status": "COMMITTED"
      }
    ],
    "link": "http://rest-api:8008/batch_statuses?id=fc9d35c7ae17191f02705a166cee84475f0d2d462e9e1e9cf637debbb3aca17c2a8c44d08c4c18ef51f6a4a4b447ce26d9d4f4f9e4df32155e671d8d744fba8d"
  }

**6. check for handling of duplicate sender_ref**

The transaction processor will fail if a message with the same sender_ref as an existing message is sent to the validator. If you submit another transaction with the same sender ref you should see a response like this:

:: 

  {
    "data": [
      {
        "id": "d6565473d3ba56726b4415e942da3d5fb6e5060a139c264ccdcd5019b4b7eaf2543951cd207aaa5a1128fc85089601b9f592d9626df20f9dbf8597c6afb18098",
        "invalid_transactions": [
          {
            "id": "5bd7e25e5d75fe1d33b673c853f487fd17c43ffda9db511a60fcdae1416f004d28e10de829cd95a9da4cf6d437f604a5e474d6dae65a53d926aef7ca48057b27",
            "message": "Invalid action: Message already exists: 123456789"
          }
        ],
        "status": "INVALID"
      }
    ],
    "link": "http://rest-api:8008/batch_statuses?id=d6565473d3ba56726b4415e942da3d5fb6e5060a139c264ccdcd5019b4b7eaf2543951cd207aaa5a1128fc85089601b9f592d9626df20f9dbf8597c6afb18098"
  }
