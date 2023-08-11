# SessionBuilder
Automatic Session Building program

## Module Part
```Python
from socket import *
from threading import Thread

from Crypto.PublicKey import RSA
from Crypto.Random import get_random_bytes
from Crypto.Cipher import AES
from Crypto.Cipher import PKCS1_OAEP
from Crypto.Util.Padding import pad
from Crypto.Util.Padding import unpad
from base64 import b64encode
from base64 import b64decode
import hashlib
import hmac

import logging
import traceback
import time
import os
import sys
```
## Logging Part
```Python
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s\t%(levelname)s - %(message)s",
    filename=f"Session.Log"
)
```

## Certification Part
```Python
class Certification:
    def __init__(self, RSA_KEY:bytes):
        self.KEY = RSA.import_key(RSA_KEY)
        self.RSA = PKCS1_OAEP.new(self.KEY)
    def Load(self, DATA:bytes):
        return self.RSA.decrypt(DATA)
```

## Session Part (MAIN)
```Python
class Session:
    def __init__(self, PORT:int, Chunk=1048576):
        self.PORT = int(PORT)
        self.OBJ = socket(AF_INET, SOCK_STREAM)
        self.OBJ.bind(("", self.PORT))
        self.Session = {}
        self.BREAK = False
        self.Chunk = int(Chunk)
    def start(self):
        logging.info("Starting session...")
        self.SERVER_KEY = RSA.generate(2048)
        logging.info("Session key has been created")
        try:
            logging.info("Session has been started")
            while not self.BREAK:
                try:
                    self.OBJ.listen()
                    connection, address = self.OBJ.accept()
                    logging.info(f"Client Connected: [{address[0]}]:{address[1]}")
                    certification_obj = Certification(self.SERVER_KEY.export_key())
                    connection.send(self.SERVER_KEY.public_key().export_key())
                    logging.info(f"Sending certification to -> [{address[0]}]:{address[1]}")
                    KEYTRANSFER = certification_obj.Load(connection.recv(self.Chunk))
                    logging.info(f"Received certification from -> [{address[0]}]:{address[1]}")
                    self.Session[address[0]] = {
                        "obj" : connection,
                        "addr" : address,
                        "certification" : KEYTRANSFER
                    }
                except KeyboardInterrupt:
                    self.BREAK = True
                    self.OBJ.close()
                    logging.info("Session has been closed")
                except:
                    logging.error(traceback.format_exc().replace("\n", "\n\t"))
        except KeyboardInterrupt:
            self.BREAK = True
            self.OBJ.close()
            logging.info("Session has been closed")
        except:
            logging.critical(traceback.format_exc().replace("\n", "\n\t"))
```
