## 各站点登陆实现思路

### 01 weibo-新浪微博

1. 登陆前有个预登陆请求，用于请求获取本次登录账号的加密信息；

2. 用户名和密码均进行了加密后提交；

   - 用户名：先进行`URL`编码后再进行`base64`编码

   - 密码：采用了`rsa`的对称加密，每次的加密信息包括混合的字符串，公钥等信息需要请求`ssologin.js`来获取

```python
    def _get_s_password(self, server_time, nonce, pubkey):
        """
        获取将密码加密后用于登录的字符串
        :return:
        """
        encode_password = (str(server_time) + "\t" + str(nonce) + "\n" + str(self.password)).encode("utf-8")
        public_key = rsa.PublicKey(int(pubkey, 16), int('10001', 16))
        encry_password = rsa.encrypt(encode_password, public_key)
        password = binascii.b2a_hex(encry_password)
        return password.decode()
```

3. 验证码：五位字母+数字的带干扰线验证码，需要手动输入或接入打码，或者搞一个模型来识别。

### 02 toutiao-今日头条

1. `Web`页面中，`PC`版登录存在滑动验证码。由于在本例程中避免使用`Selenium`进行操作，所以更换`User-Agent`采用移动端`Web`页面可以避免出现滑动验证码，与之代替的是图形验证码；

   ```python
   'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 10_3_1 like Mac OS X) AppleWebKit/603.1.30 (KHTML, like Gecko) Version/10.0 Mobile/14E304 Safari/602.1'
   ```

2. 用户名和密码均未进行任何加密；

3. 通过移动端`Web`页面登录生成的`Cookies`信息可以直接在`PC-Web`页面上使用。

### 03 sohu-搜狐新闻

1. 搜狐新闻在`PC-Web`页面中进行登录时会出现验证码，而切换至移动端`Web`页面时不会产生验证码，所以可以通过移动`Web`界面进行登录完美的绕过验证码；

2. 用户名未加密，密码采用`md5`进行加密后提交。

   ```python
       def _get_password_md5(password):
           """
           密码的md5字符串
           :return:
           """
           return md5(password.encode(encoding='UTF-8')).hexdigest()
   ```

### 04 ifeng-凤凰新闻

1. 采用`PC-Web`页面完成，账号密码均未做加密处理；

2. 登陆需要验证码，验证码为4位数由字母+数字组成。

   ```http
   验证码链接：https://id.ifeng.com/public/authcode
   ```

### 05 douban-豆瓣

1. 账号密码均未做任何加密处理，并且无验证码出现。

