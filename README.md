### Encrypt and decrypt in client side

```
    //Encrypt
    var message = JSON.stringify({'email':'example.com'});
    var key:any ='my_secret_password_for_test_1234'//key used in Python
    key = CryptoJS.enc.Utf8.parse(key);
    var iv = CryptoJS.lib.WordArray.random(16);
    var encrypted:any = CryptoJS.AES.encrypt(message, key, { iv: iv, mode: CryptoJS.mode.CBC});
    console.log('encrypted',encrypted.toString() );
    console.log('iv', iv.toString(CryptoJS.enc.Base64));
    
    
    var final = iv.concat(encrypted.ciphertext).toString(CryptoJS.enc.Base64);
    console.log('final',final );


    //Decrypt 1
    var decrypted:any = CryptoJS.AES.decrypt(encrypted, key, { iv: iv, mode: CryptoJS.mode.CBC});
    decrypted = decrypted.toString(CryptoJS.enc.Utf8);
    console.log('decrypted',decrypted );

    //Decrypt 2
    var rawData = atob('C8D+vNfCLJ7VJ71kqfBR1pcjNNxDXzd5uRT8eCbDtu4TVJwumCKJkQ0+mNqsnsYkwVd3RTpH43e+9zF1ipqj/Q==');
    var iv_new:any = btoa(rawData.substring(0,16));
    var crypttext = btoa(rawData.substring(16));
    console.log('newiv', iv_new)
    console.log('newiv', crypttext)
    
    var decrypted:any = CryptoJS.AES.decrypt(crypttext, key, { iv: CryptoJS.enc.Base64.parse(iv_new), mode: CryptoJS.mode.CBC});
    decrypted = decrypted.toString(CryptoJS.enc.Utf8);
    console.log('decrypted',decrypted );
```

### Python side

```
from Crypto.Cipher import AES
from Crypto.Util import Padding
import base64
import hashlib
from Crypto import Random
import json
from django.conf import settings
key = 'my_secret_password_for_test_1234'

class AESCipher(object):
    def __init__(self,key):
        self.key = key.encode('utf-8')
        
    def json_to_bytes(self,message):
        s = json.dumps(message)
        bytes_format = s.encode('utf-8')
        return  bytes_format
    
    def bytes_to_json(self,message):
        jsonified = json.loads(message)
        return jsonified
    
    def encrypt(self, message):
       iv = Random.get_random_bytes(AES.block_size)
       cipher = AES.new(self.key, AES.MODE_CBC, iv)
       data = Padding.pad(self.json_to_bytes(message), AES.block_size, 'pkcs7')
       cipher_text = cipher.encrypt(data)
       return base64.b64encode(iv + cipher_text).decode('utf-8')

    def decrypt(self, enc):
       enc = base64.b64decode(enc.encode('utf-8'))
       iv = enc[:AES.block_size]
       cipher = AES.new(self.key, AES.MODE_CBC, iv)
       data = Padding.unpad(cipher.decrypt(enc[AES.block_size:]), AES.block_size, 'pkcs7')
       return self.bytes_to_json(data)
   
raw_data = {'email':'example.com'}
cipher = AESCipher(key)

encrypted_text = cipher.encrypt(raw_data)
print(encrypted_text)
decrypted_json = cipher.decrypt(encrypted_text)
decrypted_json = cipher.decrypt('4/1KkOp0/7y3/TsAU3s7S3M+kWpLAdP/erNcBVVHo8AT4DvB4WkZQ7V9ZJxQswAkzjs6QZcC7mdR98CTZroFBw==')
print(decrypted_json)
```
