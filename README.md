#写在之前
项目中用到的函数简介：https://github.com/EveandBob/Introduction-to-some-functions-in-elliptic-curves-not-projects-

# 项目名称
二方签名的网络实现

# 项目实现
![Screenshot 2022-07-31 133241](https://user-images.githubusercontent.com/104854836/182011772-a04afecd-1785-4773-9d3c-7baac9893760.jpg)

这里面我使用的网络链接代码库为socket

在整个代码的实现中要注意一个事情，就是socket的传送只能传送编码后的字符串，这就意味这在传送点的坐标时在接收方需要将字符串恢复成列表，这样我采用了json.loads()函数(来自json第三方库)

# 部分代码
1.A
```python
def A():
    IDA = 0x414C494345313233405941484F4F2E434F4D
    msg = "message digest"
    IDB = 0x414C494345313233405941484F4F2E434F4C
    HOST = "127.0.0.1"
    PORT = 5005
    P = 0, 0
    Ad1 = 0
    r, s2, s3 = 0, 0, 0
    k1 = 0
    s=0
    def A_1():
        nonlocal Ad1
        Ad1=random.randrange(1,n)
        P1=calculate_np(Gx,Gy,Ad1,a,b,p)
        return P1

    def A_3():
        nonlocal k1
        M=msg.encode()
        ENTLA = get_bitsize(IDA) * 8
        ENTLB = get_bitsize(IDB) * 8
        data = ENTLA.to_bytes(2, byteorder='big', signed=False) + int_to_bytes(IDA) + ENTLB.to_bytes(2, byteorder='big', signed=False) + int_to_bytes(IDB)+int_to_bytes(a) + int_to_bytes(
            b) + int_to_bytes(Gx) + int_to_bytes(Gy) + int_to_bytes(P[0]) + int_to_bytes(P[1])
        ZA = int(sm3.sm3_hash(list(data)), 16)
        M_ = int_to_bytes(ZA) + M
        e = int(sm3.sm3_hash(list(M_)), 16)
        k1=random.randrange(1,n)
        Q1=calculate_np(Gx,Gy,k1,a,b,p)
        return Q1,e

    def A_5():
        nonlocal s
        s=((Ad1*k1)*s2+Ad1*s3-r)%n

    s = socket.socket()         # 创建 socket 对象
    host = socket.gethostname() # 获取本地主机名
    port = 12345                # 设置端口号
 
    s.connect((host, port))
    p1=str(A_1())
    s.send(p1.encode())
    P_str = s.recv(1024)
    P = json.loads(P_str.decode())
    Q1,e=A_3()
    s.send(str(Q1).encode())
    s.send(str(e).encode())
    msg=s.recv(1024)
    r,s2,s3 = json.loads(msg.decode())
    A_5()
    return r, s
```
2.B
```python
def B():
    Q1 = 0, 0
    e = 0
    Bd2 = 0
    r, s2, s3 = 0, 0, 0
    def B_1_2(P1):
        nonlocal Bd2
        Bd2 = random.randrange(1, n)
        P2 = calculate_np(P1[0], P1[1], get_inverse(Bd2, n), a, b, p)
        G_T = calculate_Tp(Gx, Gy, a, b, p)
        P = calculate_p_q(P2[0], P2[1], G_T[0], G_T[1], a, b, p)
        return P

    def B_4():
        nonlocal r,s2,s3
        k2 = random.randrange(1, n)
        Q2 = calculate_np(Gx, Gy, k2, a, b, p)
        k3 = random.randrange(1, n)
        temp=calculate_np(Q1[0],Q1[1],k3,a,b,p)
        x1,y1=calculate_p_q(temp[0],temp[1],Q2[0],Q2[1],a,b,p)
        r=(x1+e)%n
        s2=(Bd2*k3)%n
        s3=(Bd2*(r+k2))%n

    s = socket.socket()         # 创建 socket 对象
    host = socket.gethostname() # 获取本地主机名
    port = 12345                # 设置端口
    s.bind((host, port))        # 绑定端口
    s.listen(5)# 等待客户端连接
    c,addr = s.accept() # 建立客户端连接
    p1_str = c.recv(1024)
    p1=json.loads(p1_str.decode())
    P=str(B_1_2(p1))
    c.send(P.encode())
    Q1_str= c.recv(1024)
    e_str= c.recv(1024)
    Q1=json.loads(Q1_str.decode())
    e=int(e_str.decode())
    B_4()
    c.send(str([r,s2,s3]).encode())
```

# 实验结果
![Screenshot 2022-07-31 133913](https://user-images.githubusercontent.com/104854836/182011925-7478acf8-6298-4922-be6c-3a23b165bffe.jpg)

