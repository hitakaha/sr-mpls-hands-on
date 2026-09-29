# 01. SR-MPLS 基本設定

## 1.1 ループバックアドレスの確認

!!! abstract "ゴール"

    全てのルータのループバックアドレスを確認し、他のループバックを学習していないことを確認します。

ループバックアドレスの確認を行います（C8102-1 ~ C8102-5）

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show run int Loopback 0</strong>
Fri Dec 12 07:46:31.923 UTC
interface Loopback0
 ipv4 address <mark>1.1.1.1</mark> 255.255.255.255
!</code></pre>

他のルータのループバックを学習していないことを確認します（C8102-1 ~ C8102-5）

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show ip route</strong>
Fri Dec 12 07:47:10.659 UTC
(--- omitted ---)
Gateway of last resort is not set

<mark>L    1.1.1.1/32 is directly connected, 1d06h, Loopback0</mark>
C    10.14.0.0/24 is directly connected, 1d06h, HundredGigE0/0/0/0
L    10.14.0.1/32 is directly connected, 1d06h, HundredGigE0/0/0/0
C    10.15.0.0/24 is directly connected, 1d06h, HundredGigE0/0/0/1
L    10.15.0.1/32 is directly connected, 1d06h, HundredGigE0/0/0/1
C    198.18.10.0/24 is directly connected, 1d06h, HundredGigE0/0/0/2
L    198.18.10.51/32 is directly connected, 1d06h, HundredGigE0/0/0/2
C    198.18.128.0/18 is directly connected, 1d06h, MgmtEth0/RP0/CPU0/0
L    198.18.134.51/32 is directly connected, 1d06h, MgmtEth0/RP0/CPU0/0
RP/0/RP0/CPU0:c8102-1#</code></pre>

## 1.2 OSPF 設定

!!! abstract "ゴール"

    OSPF を設定し、全てのノード間でループバックによる疎通を確立します。

OSPF の設定を行います。（C8102-1 ~ C8102-5）

- router-id は、設定対象ルータのループバックアドレス (1.1.1.1, 2.2.2.2, etc.) に<br>
変更してください。
- OSPF を有効にするインタフェースは、ルータに応じて変更してください。

<pre class="cli"><code>router ospf core
 router-id <mark>1.1.1.1</mark>
 area 0
  interface Loopback0
   passive enable
  !
  interface <mark>HundredGigE0/0/0/0</mark>
   network point-to-point
  !
  interface <mark>HundredGigE0/0/0/1</mark>
   network point-to-point
  !
 !
!</code></pre>

OSPF のステート確認を行います（C8102-1 ~ C8102-5）

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show ospf neighbor</strong> 
Mon Dec 15 00:52:06.609 UTC

* Indicates MADJ interface
# Indicates Neighbor awaiting BFD session up

Neighbors for OSPF core

Neighbor ID     Pri   State           Dead Time   Address         Interface
4.4.4.4         1     <mark>FULL</mark>/  -        00:00:31    10.14.0.4       HundredGigE0/0/0/0
    Neighbor is up for 2d16h
5.5.5.5         1     <mark>FULL</mark>/  -        00:00:31    10.15.0.5       HundredGigE0/0/0/1
    Neighbor is up for 2d16h</code></pre>

ルーティングテーブルの確認を行います。（C8102-1 ~ C8102-5）

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show ip route</strong>
Mon Dec 15 00:53:49.786 UTC

Gateway of last resort is not set

L    1.1.1.1/32 is directly connected, 3d23h, Loopback0
<mark>O    2.2.2.2/32 [110/3] via 10.14.0.4, 2d16h, HundredGigE0/0/0/0</mark>
<mark>O    3.3.3.3/32 [110/3] via 10.15.0.5, 2d16h, HundredGigE0/0/0/1</mark>
<mark>O    4.4.4.4/32 [110/2] via 10.14.0.4, 2d16h, HundredGigE0/0/0/0</mark>
<mark>O    5.5.5.5/32 [110/2] via 10.15.0.5, 2d16h, HundredGigE0/0/0/1</mark>
C    10.14.0.0/24 is directly connected, 3d23h, HundredGigE0/0/0/0
L    10.14.0.1/32 is directly connected, 3d23h, HundredGigE0/0/0/0
C    10.15.0.0/24 is directly connected, 3d23h, HundredGigE0/0/0/1
L    10.15.0.1/32 is directly connected, 3d23h, HundredGigE0/0/0/1
O    10.23.0.0/24 [110/3] via 10.14.0.4, 2d16h, HundredGigE0/0/0/0
                  [110/3] via 10.15.0.5, 2d16h, HundredGigE0/0/0/1
O    10.24.0.0/24 [110/2] via 10.14.0.4, 2d16h, HundredGigE0/0/0/0
O    10.35.0.0/24 [110/2] via 10.15.0.5, 2d16h, HundredGigE0/0/0/1
O    10.45.0.0/24 [110/2] via 10.14.0.4, 2d16h, HundredGigE0/0/0/0
                  [110/2] via 10.15.0.5, 2d16h, HundredGigE0/0/0/1
C    198.18.10.0/24 is directly connected, 3d23h, HundredGigE0/0/0/2
L    198.18.10.51/32 is directly connected, 3d23h, HundredGigE0/0/0/2
C    198.18.128.0/18 is directly connected, 3d23h, MgmtEth0/RP0/CPU0/0
L    198.18.134.51/32 is directly connected, 3d23h, MgmtEth0/RP0/CPU0/0
RP/0/RP0/CPU0:c8102-1#</code></pre>

## 1.3 SR-MPLS 設定

!!! abstract "ゴール"

    SR-MPLS を設定し、ループバック通信がラベルで行われることを確認します。

LFIB テーブルに何も表示されないことを確認します。（C8102-1 ~ C8102-5）

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show mpls forwarding</strong> 
Mon Dec 15 01:04:29.527 UTC
RP/0/RP0/CPU0:c8102-1#</code></pre>

SR-MPLS の有効化とPrefix SID 設定を行います。（C8102-1 ~ C8102-5）

- Prefix-SID Indexは、ルータ番号と一致した値を設定してください。<br>
例えば、C8102-1なら１、C8102-2なら２

<pre class="cli"><code>router ospf core
 segment-routing mpls
 area 0
  interface Loopback0
   prefix-sid index <mark>1</mark>
  !
 !
!</code></pre>

LFIB テーブルにて、他ルータの Prefix SID (Pfx) 及び自身の Adjacency SID (Adj) が存在することを確認します。（C8102-1 ~ C8102-5）

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show mpls forwarding</strong> 
Mon Dec 15 01:29:34.470 UTC
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes       
Label  Label       or ID              Interface                    Switched    
------ ----------- ------------------ ------------ --------------- ------------
<mark>16002  16002       SR Pfx (idx 2)     Hu0/0/0/0    10.14.0.4       0           </mark>
<mark>16003  16003       SR Pfx (idx 3)     Hu0/0/0/1    10.15.0.5       0           </mark>
<mark>16004  Pop         SR Pfx (idx 4)     Hu0/0/0/0    10.14.0.4       0           </mark>
<mark>16005  Pop         SR Pfx (idx 5)     Hu0/0/0/1    10.15.0.5       0           </mark>
<mark>24000  Pop         SR Adj (idx 0)     Hu0/0/0/0    10.14.0.4       0           </mark>
<mark>24001  Pop         SR Adj (idx 0)     Hu0/0/0/1    10.15.0.5       0</mark>           
RP/0/RP0/CPU0:c8102-1#</code></pre>

C8102-4でLFIBテーブルを確認すると、C8102-3向けのルートがECMPになっていることがわかります。

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-4#<strong>show mpls forwarding </strong>
Mon Dec 22 05:09:46.249 UTC
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes       
Label  Label       or ID              Interface                    Switched    
------ ----------- ------------------ ------------ --------------- ------------
16001  Pop         SR Pfx (idx 1)     Hu0/0/0/2    10.14.0.1       0           
16002  Pop         SR Pfx (idx 2)     Hu0/0/0/1    10.24.0.2       0           
<mark>16003  16003       SR Pfx (idx 3)     Hu0/0/0/0    10.45.0.5       0           </mark>
<mark>       16003       SR Pfx (idx 3)     Hu0/0/0/1    10.24.0.2       0</mark>           
16005  Pop         SR Pfx (idx 5)     Hu0/0/0/0    10.45.0.5       0           
24000  Pop         SR Adj (idx 0)     Hu0/0/0/0    10.45.0.5       0           
24001  Pop         SR Adj (idx 0)     Hu0/0/0/1    10.24.0.2       0           
24002  Pop         SR Adj (idx 0)     Hu0/0/0/2    10.14.0.1       0           
RP/0/RP0/CPU0:c8102-4#</code></pre>

CEF 確認を行います。（C8102-1 ~ C8102-5）

下記の例では、C8102-1ではC8102-2(2.2.2.2/32) 宛のパケットには Label 16002 が付加されることがわかります。

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show cef </strong><strong><mark>2.2.2.2/32</mark></strong>
Mon Dec 15 01:45:34.437 UTC
2.2.2.2/32, version 68, labeled SR, internal 0x1000001 0x8310 (ptr 0x9c21d1a8) [1], 0x600 (0x9c1ea278), 0xa28 (0x9b425368)
 Updated Dec 15 01:42:31.562 
 local adjacency to HundredGigE0/0/0/0

 Prefix Len 32, traffic index 0, precedence n/a, priority 1, encap-id 0x1000b00000001
  gateway array (0x9c083a98) reference count 3, flags 0x68, source rib (7), 0 backups
                [2 type 5 flags 0x8401 (0x9b464838) ext 0x0 (0x0)]
  LW-LDI[type=5, refc=3, ptr=0x9c1ea278, sh-ldi=0x9b464838]
  gateway array update type-time 1 Dec 15 01:42:31.562
 LDI Update time Dec 15 01:42:31.562
 LW-LDI-TS Dec 15 01:42:31.562
   via 10.14.0.4/32, HundredGigE0/0/0/0, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 0 NHID 0x1 [0x9d487be0 0x0]
    next hop 10.14.0.4/32
    local adjacency
     local label 16002      <mark>labels imposed {16002}</mark>

    Load distribution: 0 (refcount 2)

    Hash  OK  Interface                 Address
    0     Y   HundredGigE0/0/0/0        10.14.0.4      
RP/0/RP0/CPU0:c8102-1#</code></pre>

Traceroute 確認を実施します。（C8102-1 ~ C8102-5）

下記の例では、C8102-1からC8102-2(2.2.2.2/32) 宛に Traceroute を実施しており、Label 16002 が付加され、Next Hopである C8102-4 へ転送されていることがわかります。

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<mark>traceroute 2.2.2.2</mark>
Mon Dec 15 01:34:11.126 UTC
Type escape sequence to abort.
Tracing the route to 2.2.2.2
 1  10.14.0.4 [<mark>MPLS: Label 16002 Exp 0</mark>] 16 msec  7 msec  7 msec 
 2  10.24.0.2 9 msec  *  9 msec 
RP/0/RP0/CPU0:c8102-1#</code></pre>

## 1.4 SRGB の変更（オプション）

!!! abstract "ゴール"

    SRGB の変更を行い、OSPF でどのようにアドバタイズされるかを理解します。

SRGB のデフォルト設定の確認を行います。（C8102-1 ~ C8102-5）

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show mpls label table detail</strong>
Mon Dec 15 01:30:22.014 UTC
Table Label   Owner                           State  Rewrite
----- ------- ------------------------------- ------ -------
0     0       LSD(A)                          InUse  Yes
0     1       LSD(A)                          InUse  Yes
0     2       LSD(A)                          InUse  Yes
0     13      LSD(A)                          InUse  Yes
0     16000   OSPF(A):ospf-core               InUse  No
  <mark>(Lbl-blk SRGB, vers:0, (start_label=16000, size=8000)</mark>
0     24000   OSPF(A):ospf-core               InUse  Yes
  (SR Adj Segment IPv4, vers:0, index=0, type=2, intf=Hu0/0/0/0, nh=10.14.0.4)
0     24001   OSPF(A):ospf-core               InUse  Yes
  (SR Adj Segment IPv4, vers:0, index=0, type=2, intf=Hu0/0/0/1, nh=10.15.0.5)
RP/0/RP0/CPU0:c8102-1#</code></pre>

OSPF での Prefix-SID の広報を確認します。（参考）

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show ospf database opaque-area 7.0.0.1 self-originate</strong>
Mon Dec 15 01:32:28.149 UTC


            OSPF Router with ID (1.1.1.1) (Process ID core)

                Type-10 Opaque Link Area Link States (Area 0)

  LS age: 383
  Options: (No TOS-capability, DC)
  LS Type: Opaque Area Link
  Link State ID: 7.0.0.1
  Opaque Type: 7
  Opaque ID: 1
  Advertising Router: 1.1.1.1
  LS Seq Number: 80000001
  Checksum: 0xe5b0
  Length: 44

    Extended Prefix TLV: Length: 20
      Route-type: 1
      AF        : 0
      Flags     : 0x40
      <mark>Prefix    : 1.1.1.1/32</mark>

      SID sub-TLV: Length: 8
        Flags     : 0x0
        MTID      : 0
        Algo      : 0
        <mark>SID Index : 1</mark>

RP/0/RP0/CPU0:c8102-1#</code></pre>

OSPF での SRGB の広報の確認を行います。（オプション）

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show ospf database opaque-area 4.0.0.0 self-originate</strong>
Mon Dec 15 01:33:16.694 UTC


            OSPF Router with ID (1.1.1.1) (Process ID core)

                Type-10 Opaque Link Area Link States (Area 0)

  LS age: 1460
  Options: (No TOS-capability, DC)
  LS Type: Opaque Area Link
  Link State ID: 4.0.0.0
  Opaque Type: 4
  Opaque ID: 0
  Advertising Router: 1.1.1.1
  LS Seq Number: 80000002
  Checksum: 0x2b7d
  Length: 72

    Router Information TLV: Length: 4
    Capabilities:
      Graceful Restart Helper Capable
      Stub Router Capable
      All capability bits: 0x60000000

    Segment Routing Algorithm TLV: Length: 2
      Algorithm: 0
      Algorithm: 1

    Segment Routing Range TLV: Length: 12
      <mark>Range Size: 8000</mark>

        SID sub-TLV: Length 3
         <mark>Label: 16000</mark>

    Node MSD TLV: Length: 2
        Type: 1, Value 8

    Dynamic Hostname TLV: Length: 7
      Hostname: c8102-1

RP/0/RP0/CPU0:c8102-1#</code></pre>

SRGB の設定変更を実施します。（C8102-1 ~ C8102-5）

全てのルータに同じ設定を行います。

<pre class="cli"><code>segment-routing
 global-block 16000 19999
!</code></pre>

SRGB 確認を行います。（C8102-1 ~ C8102-5）

SRGBのサイズが8,000から4,000に変更されていることがわかります。

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show mpls label table detail</strong>
Mon Dec 15 01:42:51.997 UTC
Table Label   Owner                           State  Rewrite
----- ------- ------------------------------- ------ -------
0     0       LSD(A)                          InUse  Yes
0     1       LSD(A)                          InUse  Yes
0     2       LSD(A)                          InUse  Yes
0     13      LSD(A)                          InUse  Yes
0     15000   LSD(A)                          InUse  No
  (Lbl-blk SRLB, vers:0, (start_label=15000, size=1000, app_notify=0)
0     16000   OSPF(A):ospf-core               InUse  No
  <mark>(Lbl-blk SRGB, vers:0, (start_label=16000, size=4000)</mark>
0     24000   OSPF(A):ospf-core               InUse  Yes
  (SR Adj Segment IPv4, vers:0, index=0, type=2, intf=Hu0/0/0/0, nh=10.14.0.4)
0     24001   OSPF(A):ospf-core               InUse  Yes
  (SR Adj Segment IPv4, vers:0, index=0, type=2, intf=Hu0/0/0/1, nh=10.15.0.5)
RP/0/RP0/CPU0:c8102-1#</code></pre>

OSPF での SRGB の広報を確認します。（オプション）

<pre class="cli"><code>RP/0/RP0/CPU0:c8102-1#<strong>show ospf database opaque-area 4.0.0.0 self-originate</strong>
Mon Dec 15 01:44:13.604 UTC


            OSPF Router with ID (1.1.1.1) (Process ID core)

                Type-10 Opaque Link Area Link States (Area 0)

  LS age: 148
  Options: (No TOS-capability, DC)
  LS Type: Opaque Area Link
  Link State ID: 4.0.0.0
  Opaque Type: 4
  Opaque ID: 0
  Advertising Router: 1.1.1.1
  LS Seq Number: 80000004
  Checksum: 0x3039
  Length: 88

    Router Information TLV: Length: 4
    Capabilities:
      Graceful Restart Helper Capable
      Stub Router Capable
      All capability bits: 0x60000000

    Segment Routing Algorithm TLV: Length: 2
      Algorithm: 0
      Algorithm: 1

    Segment Routing Range TLV: Length: 12
      <mark>Range Size: 4000</mark>

        SID sub-TLV: Length 3
         <mark>Label: 16000</mark>

    Node MSD TLV: Length: 2
        Type: 1, Value 8

    Segment Routing Local Block TLV: Length: 12
      Range Size: 1000

        SID sub-TLV: Length 3
         Label: 15000

    Dynamic Hostname TLV: Length: 7
      Hostname: c8102-1

RP/0/RP0/CPU0:c8102-1#</code></pre>
