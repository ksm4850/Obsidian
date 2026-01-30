
https://malachai.tistory.com/121

클라이언트에서 접속
```
sudo openvpn --config my-client.ovpn
```

연결된 ip 확인
```bash
#sudo cat /var/log/openvpn/ipp.txt
sudo cat /var/log/openvpn/openvpn-status.log

```

개인 키 생성
```bash
cd ~/EasyRSA-3.0.8
# 키 생성
./easyrsa gen-req {파일명} nopass

# 서버키로 검증
./easyrsa sign-req client {파일명}

# 생성된 파일 이동
cp pki/private/my-client.key ~/client-configs/keys
# crt 인증서 이동
cp pki/issued/{파일명}.crt ~/client-configs/keys/
```
 

인증서/키 취소
```bash
# 인증서 폐기
./easyrsa revoke 클라이언트명

# CRL 생성
./easyrsa gen-crl

# CRL 파일 복사
# CRL파일을 openvpn 서버가 읽을 수 있는 위치로 복사
cp pki/crl.pem /etc/openvpn/server/

# 서버 설정에 CRL 경로 추가
crl-verify /etc/openvpn/server/crl.pem

# 서비스 재시작
systemctl restart openvpn@server
```

연결된 클라 확인
```
sudo cat /var/log/openvpn/openvpn-status.log
```