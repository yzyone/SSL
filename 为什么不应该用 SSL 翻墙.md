# 为什么不应该用 SSL 翻墙 #

SSL 设计目标:

- 防内容篡改
- 防冒充服务器身份
- 加密通信内容

而翻墙的目标:

- 不被检测出客户端在访问什么网站
- 不被检测出服务器在提供翻墙服务

SSL 和这个目标还是有一些出入。其中最大的问题是，2. 防冒充服务器身份 这个功能多余了。他会导致墙嗅探出证书信息，继而墙会知道服务器身份。如果墙知道一个服务器身份是用来翻墙的，它要做的仅仅是封掉使用这个证书的所有 IP。

墙看见的 SSL 握手响应头部如下：

```
QMQڑ????β????!AFf?a?1??[?
                         ? G
0???R???̙G!GfX-??5?k ??Q?
                        u

                         q
0??1    *?H??             n?0?|0?d?"?7??^??eĵ??F0
    0                                                          UUS10U
VeriSign, Inc.10U
                 VeriSign Trust Network1;09U
140510235959Z0?10                           2Terms of use at https://www.verisign.com/rpa (c)061402U+VeriSign Class 3 Extended Validation SSL CA0
                 +?7<US10
                         +?7Delaware10Private Organization10U43374461
                                                                     0    UUS10
                                                                             U9410710U
San Francisco1!0U	795 Folsom St, Suite 60010U
Twitter, Inc.10U
                Twitter Security10U
?0?     *?H??                      twitter.com0?"0
???w??tSmZ	?T#7Tn,?l\C????
                               ??^d?L?*q?	???\n*???*?X?[?????-?b?V?Ic[[ݙn??D??i;?j~,亵Ȯu?z۴Jz9qr??כ????G?#???u2j?7_;?3[Y?&?P???P?@?U?N?,?Tx?e?N???SVߝ???*!Po???Q???!??sIpҒ#`H??U'?"^?*?7
_?>???Z|`W?.p_??????p????v2?p????0?0'U 0?www.twitter.com?
                                                         twitter.com0	U00U?x?Fy?n?]@H?G???(?1?0
                                                                                                 U?0BU;0907?5?3?1http://EVSecure-crl.verisign.com/EVSecure2006.crl0DU =0;09
                                                                                                                                                                           `?H??E0*+https://www.verisign.com/rpa0U%+0U#0???P???%Z{U?O?c??XkC0+p0n0+0?!http://EVSecure-ocsp.verisign.com0+0?1http://EVSecure-aia.verisign.com/EVSecure2006.cer0+
                                                                                                                                                          b0`?^?\0Z0X0V	image/gif0!00+Kk?(?
?     ??*?H??K?!0&$http://logo.verisign.com/vslogo1.gif0
??Z
   GD?𥛄8ݫ?NJnJXn?????]?? -";J?#O????
                                     <r?Q&?.??;? F
?????K??Fq?mD?-?Sl??=3???]?.??=v?I???
???ѱ??)i?Y!K?L???GC??rI0IR??f!~
b4˼??]?b?????_5a=?-B¹ax???rif߰?j?[??j?????3???d?Ўo
kS
?m'?9????X?H???`ln?n%???ߍ5G?H?!w?+x:
?u?86K?[,!?z?ipŵh?4uf????/N? Dt?'\??d+A???]??	??gqP???V{?0c?1??mKǫ??o;
```

关于对抗 GFW 的特征检测

不论是 Shadowsocks 还是 VPN 或 SSH 隧道，共性都很明显——长时间的 TCP 长连接。
这样一种特征，非常明显是被用来翻墙。

或许可以考虑换一种思路：

用普通的，明文的 HTTP 请求来承载上层的翻墙数据。

以 SOCKS 代理为例：

把 SOCKS 代理切割成两个部分，一部分部署在客户端，一部分部署在 VPS。

两者之间的通讯，伪装成普通的，明文的 HTTP 流量。

（表面上看，像是明文的，但是承载的数据，依然用加密方式，比如打包成图片的内容）

这时候，SOCKS代理的客户端部分，对本地开启SOCKS 监听端口，然后把数据包装成 HTTP 发送到 VPS 的服务端。VPS 的部分再解包，重新转化为原始的 SOCKS 流量发送到最终目的 IP

对本地的网络软件而言，看到的依然是普通的 SOCKS 代理，因此浏览器或 IM 可以无缝兼容。

这样的流量特征，就非常类似于平时访问网站的流量特征。可以避免被 GFW 察觉。
如果伪装得好，即使是 Ghost Assassin 在 7单元 提到的“人工审核”，也可以骗过。


SSL是使用RSA来进行密钥交换的，说白了就是客户端生成一个动态的加密密钥匙用服务器的Pubkey加密发给服务器，服务器用Private Key解密得到这个加密密钥，在密钥交换过程中会暴露服务器证书，特别是域名/IP。

不了解RSA加密原理的可以看一下我的这篇文章：
https://cyyself.name/2017/07/rsa-algorithm/

对于SS服务器来说，采用预共享密钥的加密方式免去了RSA密钥交换环节，抓不到任何有明显特征的握手包，减少了特征。

至于SSL识别对于GFW有多容易，参考OpenVPN。

如果客户端预存服务器的证书(公钥)，跳过通过网络获取服务器证书这一步，是否可以减少大量的握手特征？

用于翻墙的https代理服务器，其实不需要CA，一个自签署的证书就足够了，而且公钥可以根本不公开。

当然这改造版的https协议不再是一个标准的https协议，实现起来不能用现有的各种包。

原文链接：https://gist.github.com/clowwindy/5947691
