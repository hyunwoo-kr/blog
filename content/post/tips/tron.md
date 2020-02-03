---
title: "[tips] TRON - Python API를 사용한 송금"
date: 2020-02-03T17:51:02+09:00
categories:
- tips
tags:
- tron
- tronapi
- tron-api-python
- self.tron.private_key
keywords:
- tech
drift : true
#thumbnailImage: //example.com/image.jpg
---
Tron Python API 모듈을 사용해서 TRC10 '송금'을 하려면 어떻게 해야 할까?
<!--more-->
[참고 사이트]
- [TRON Developers](https://developers.tron.network/)
- [TRC10 이란?](https://developers.tron.network/docs/trc10-token)

GitHub의 [Python API](https://github.com/iexbase/tron-api-python/blob/master/examples/send-transaction.py)를 그대로 따라하다 보면 아래와 같은 error가 발생한다.
## Github에서 제공하는 TRC10  송금

```python
from tronapi import Tron
from tronapi import HttpProvider

full_node = HttpProvider('https://api.trongrid.io')
solidity_node = HttpProvider('https://api.trongrid.io')
event_server = HttpProvider('https://api.trongrid.io')

tron = Tron(full_node=full_node,
            solidity_node=solidity_node,
            event_server=event_server)


tron.private_key = 'private_key'
tron.default_address = 'default address'

# added message
send = tron.trx.send_transaction('to', 1)

print(send)
```




## 오류 내용
```
Traceback (most recent call last):
  .....
    send = tron.trx.send_token(to, amount, token_id)
  File ".../.venv/lib/python3.6/site-packages/tronapi/trx.py", line 458, in send_token
    sign = self.sign(tx)
  File ".../.venv/lib/python3.6/site-packages/tronapi/trx.py", line 574, in sign
    address = self.tron.address.from_private_key(self.tron.private_key).hex.lower()
  File ".../.venv/lib/python3.6/site-packages/tronapi/common/account.py", line 59, in from_private_key
    return PrivateKey(private_key).address
  File ".../.venv/lib/python3.6/site-packages/tronapi/common/account.py", line 70, in __init__
    _private = unhexlify(bytes(private_key, encoding='utf8'))
TypeError: encoding without a string argument
```

## 오류 문구 "TypeError: encoding without a string argument"
이 에러가 발생하는 이유가 결론부터 보면, 아래 줄이 문제이고

```python
address = self.tron.address.from_private_key(self.tron.private_key).hex.lower()
```
그 중에서도
### self.tron.private_key 의 값이 None 이라서 그렇다


제공해 주는 위의 소스에서 아래 항목을 추가 하면 해결 된다.

```python
tron.trx.tron.private_key = 'private_key'
```

수정 된 전체 소스응 아래와 같다
```
from tronapi import Tron
from tronapi import HttpProvider

full_node = HttpProvider('https://api.trongrid.io')
solidity_node = HttpProvider('https://api.trongrid.io')
event_server = HttpProvider('https://api.trongrid.io')

tron = Tron(full_node=full_node,
            solidity_node=solidity_node,
            event_server=event_server)


tron.private_key = 'private_key'
tron.default_address = 'default address'

# added by hyunwoo.
tron.trx.tron.private_key = 'private_key'

# added message
send = tron.trx.send_transaction('to', 1)

print(send)
```

[결론]
 - 해결하고 보니, TRON Python API는 뭔가 만들다 만 느낌이 든다. 