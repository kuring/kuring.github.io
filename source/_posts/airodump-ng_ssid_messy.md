---
title: 解决airodump-ng显示ssid名称的乱码问题
Status: public
url: airodump-ng_ssid_messy
date: 2015-05-04
toc: yes
---

# 问题描述

无线wifi的essid支持英文和中文，中文的编码在802.11协议并没有规定，对于802.11协议而言仅将essid看作是二进制。而中文又存在多种编码方式，最常见的就是GB18030（我这里直接用GB18030代替了GB系列的字符集）和UTF-8了。

iwlist程序通过命令`iwlist wlan0 scanning`可以在终端上正常显示UTF-8编码的essid，对于其他编码的中文仍然是乱码，这也就非常容易理解了。因为具体的essid能否将中文正常显示在终端屏幕上跟essid的编码和当前终端环境的编码是否能够匹配有关，如果essid的编码和当前终端环境的编码均为UTF-8，则essid可以在屏幕上正常显示。如果当前网络中的可以搜索到的essid即包含了GB18030编码又包含了UTF-8编码，则打印在终端上的essid必然会有乱码的情况出现。

# airodump-ng程序问题

对于airodump-ng程序而言，即时是essid的编码和终端编码一致也会出现某些中文字符乱码的问题，这一点比较奇怪。比如“免费”中的“免”字是乱码，“费”却能正常显示。通过这一现象有理由怀疑airodump-ng对essid做了某些处理。

经过查看源码发现，在airodump-ng.c文件中存在三处如下类似代码，作用为将essid中的ascii值在(126,160)之间的转换为"."。看来airodump-ng程序并没有考虑到中文的情况，仅将ascii中无法显示的字符做了转换。将程序中的三处代码注释后就可以正常显示了。具体三处代码可以通过搜索'.'来查找。

```c++
for( i = 0; i < n; i++ )
{
	c = p[2 + i];
	if( c == 0 || ( c > 126 && c < 160 ) )
	{
		c = '.';  //could also check ||(c>0 && c<32)
	}
	st_cur->probes[st_cur->probe_index][i] = c;
}
```

# NetworkManager

通过实践发现，GNOME和KDE桌面下的查看无线网络连接的ssid是可以正常显示的，即可以正常显示GB18030，又可以正常显示UTF-8编码的essid。则可以推测，在桌面环境下的搜索网络的程序肯定对编码做了某些处理，顺着这个思路，就可以查找GNOME或KDE的代码了。

在GNOME的源码中看到了network-manager-applet，该程序即为桌面上查看无线网络连接的小控件。在applet-device-wifi.c文件中看到了如下代码，其中的`nm_utils_ssid_to_utf8`函数即为将其他编码转换为UTF-8编码的函数。

```c++
static char *
get_ssid_utf8 (NMAccessPoint *ap)
{
	char *ssid_utf8 = NULL;
	const GByteArray *ssid;

	if (ap) {
		ssid = nm_access_point_get_ssid (ap);
		if (ssid)
			ssid_utf8 = nm_utils_ssid_to_utf8 (ssid);
	}
	if (!ssid_utf8)
		ssid_utf8 = g_strdup (_("(none)"));

	return ssid_utf8;
}
```

`nm_utils_ssid_to_utf8`函数定义在NetworkManager工程中的nm-utils.c文件中。该函数的代码如下，该函数具体功能可以查看代码中的注释，已经非常详细了。其中以`g_`开头的函数是glib库中的函数。

```c++
char *
nm_utils_ssid_to_utf8 (const GByteArray *ssid)
{
	char *converted = NULL;
	char *lang, *e1 = NULL, *e2 = NULL, *e3 = NULL;

	g_return_val_if_fail (ssid != NULL, NULL);

	if (g_utf8_validate ((const gchar *) ssid->data, ssid->len, NULL))
		return g_strndup ((const gchar *) ssid->data, ssid->len);

	/* LANG may be a good encoding hint */
	g_get_charset ((const char **)(&e1));
	if ((lang = getenv ("LANG"))) {
		char * dot;

		lang = g_ascii_strdown (lang, -1);
		if ((dot = strchr (lang, '.')))
			*dot = '\0';

		get_encodings_for_lang (lang, &e1, &e2, &e3);
		g_free (lang);
	}

	converted = g_convert ((const gchar *) ssid->data, ssid->len, "UTF-8", e1, NULL, NULL, NULL);
	if (!converted && e2)
		converted = g_convert ((const gchar *) ssid->data, ssid->len, "UTF-8", e2, NULL, NULL, NULL);

	if (!converted && e3)
		converted = g_convert ((const gchar *) ssid->data, ssid->len, "UTF-8", e3, NULL, NULL, NULL);

	if (!converted) {
		converted = g_convert_with_fallback ((const gchar *) ssid->data, ssid->len,
		                                     "UTF-8", e1, "?", NULL, NULL, NULL);
	}

	return converted;
}
```

nm_utils_ssid_to_utf8该函数位于libnm-util.so.1动态库中，可通过`nm -D  /usr/lib64/libnm-util.so.1 | grep nm_utils_ssid_to_utf8`命令查看导出表中存在该函数。但是系统中并不存在该函数的头文件libnm-util.h，给该库的调用增加了不少难度。可以通过将相关头文件引入到该工程编译的方式来完成，但是可能会牵涉到的头文件比较多，比较繁琐。

我这里直接采用了将NetworkManager中相关代码抓取出来的思路，并将其封装成类的形式以方便调用。具体代码可以参照demo中的例子。

# glib

glib是GTK底层调用的核心库，跟glibc是没有关系的，虽然名字中仅差一个字母。为了调用该库需要在编译的时候添加*`pkg-config --cflags --libs glib-2.0`*信息，以引入需要的头文件和要链接的库。

# 相关下载

[文中用到的软件源码和程序demo](http://pan.baidu.com/s/1qWqjMCc)

# 引用

* [GNOME源码列表](https://git.gnome.org/browse/)
