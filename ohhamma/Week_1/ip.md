## IP addressing

### IP addr

![1](./images/ip-1.png)

- **32-bit** identifier associated with host / router interface
- dotted-decimal notation

### interface

![2](./images/ip-2.png)

- **connection**
- **NIC** : Network Interface Card
- router : multiple interfaces
- host : one or two interfaces


## Subnets

![3](./images/ip-3.png)

- islands of isolated networks
- interfaces, **physically** reach each other, with**out** passing router
- IP addr structure
  - `subnet` part : 네트워크 이름, high order bits(MSB부터 시작)
  - `host` part : 네트워크 내의 호스트 이름, remaining low order bits

![4](./images/ip-4.png)


## CIDR

- **C**lassless **I**nter**D**omain **R**outing
- subnet in **arbitary** length
- format: `a.b.c.d/x`
  - x: # bits of subnet, 8의 배수 아니어도됨