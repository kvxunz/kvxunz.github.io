
# **封装客户端**
## **定义**
- ftp协议最底层是tcp报文，如果从socket开始编程，工作量巨大。
- 寻找ftp客户端的开源库（ftplib），封装成简单易用的类cftp。

## **_ftp.h**
```c++
#ifndef __FTP_H
#define __FTP_H

#include "_public.h"
#include "ftplib.h"

namespace idc
{

class cftpclient
{
private:
    netbuf *m_ftpconn;   // ftp连接句柄。
public:
    unsigned int  m_size;      // 文件的大小，单位：字节。
    string m_mtime;     // 文件的修改时间，格式：yyyymmddhh24miss。

    // 以下三个成员变量用于存放login方法登录失败的原因。
    bool m_connectfailed;    // 如果网络连接失败，该成员的值为true。
    bool m_loginfailed;      // 如果登录失败，用户名和密码不正确，或没有登录权限，该成员的值为true。
    bool m_optionfailed;     // 如果设置传输模式失败，该成员变量的值为true。

    cftpclient();  // 类的构造函数。
    ~cftpclient();  // 类的析构函数。

    cftpclient(const cftpclient&) = delete;
    cftpclient& operator=(const cftpclient) = delete;

    void initdata();   // 初始化m_size和m_mtime成员变量。

    // 登录ftp服务器。
    // host：ftp服务器ip地址和端口，中间用":"分隔，如"192.168.1.1:21"。
    // username：登录ftp服务器用户名。
    // password：登录ftp服务器的密码。
    // imode：传输模式，1-FTPLIB_PASSIVE是被动模式，2-FTPLIB_PORT是主动模式，缺省是被动模式。
    bool login(const string &host,const string &username,const string &password,const int imode=FTPLIB_PASSIVE);
  
    // 注销。
    bool logout();

    // 获取ftp服务器上文件的时间。
    // remotefilename：待获取的文件名。
    // 返回值：false-失败；true-成功，获取到的文件时间存放在m_mtime成员变量中。
    bool mtime(const string &remotefilename);

    // 获取ftp服务器上文件的大小。
    // remotefilename：待获取的文件名。
    // 返回值：false-失败；true-成功，获取到的文件大小存放在m_size成员变量中。
    bool size(const string &remotefilename);

    // 改变ftp服务器的当前工作目录。
    // remotedir：ftp服务器上的目录名。
    // 返回值：true-成功；false-失败。
    bool chdir(const string &remotedir);

    // 在ftp服务器上创建目录。
    // remotedir：ftp服务器上待创建的目录名。
    // 返回值：true-成功；false-失败。
    bool mkdir(const string &remotedir);

    // 删除ftp服务器上的目录。
    // remotedir：ftp服务器上待删除的目录名。
    // 返回值：true-成功；如果权限不足、目录不存在或目录不为空会返回false。
    bool rmdir(const string &remotedir);

    // 发送NLST命令列出ftp服务器目录中的子目录名和文件名。
    // remotedir：ftp服务器的目录名。
    // listfilename：用于保存从服务器返回的目录和文件名列表。
    // 返回值：true-成功；false-失败。
    // 注意：如果列出的是ftp服务器当前目录，remotedir用"","*","."都可以，但是，不规范的ftp服务器可能有差别。
    bool nlist(const string &remotedir,const string &listfilename);

    // 从ftp服务器上获取文件。
    // remotefilename：待获取ftp服务器上的文件名。
    // localfilename：保存到本地的文件名。
    // bcheckmtime：文件传输完成后，是否核对远程文件传输前后的时间，保证文件的完整性。
    // 返回值：true-成功；false-失败。
    // 注意：文件在传输的过程中，采用临时文件命名的方法，即在localfilename后加".tmp"，在传输
    // 完成后才正式改为localfilename。
    bool get(const string &remotefilename,const string &localfilename,const bool bcheckmtime=true);

    // 向ftp服务器发送文件。
    // localfilename：本地待发送的文件名。
    // remotefilename：发送到ftp服务器上的文件名。
    // bchecksize：文件传输完成后，是否核对本地文件和远程文件的大小，保证文件的完整性。
    // 返回值：true-成功；false-失败。
    // 注意：文件在传输的过程中，采用临时文件命名的方法，即在remotefilename后加".tmp"，在传输
    // 完成后才正式改为remotefilename。
    bool put(const string &localfilename,const string &remotefilename,const bool bchecksize=true);

    // 删除ftp服务器上的文件。
    // remotefilename：待删除的ftp服务器上的文件名。
    // 返回值：true-成功；false-失败。
    bool ftpdelete(const string &remotefilename);

    // 重命名ftp服务器上的文件。
    // srcremotefilename：ftp服务器上的原文件名。
    // dstremotefilename：ftp服务器上的目标文件名。
    // 返回值：true-成功；false-失败。
    bool ftprename(const string &srcremotefilename,const string &dstremotefilename);

    // 向ftp服务器发送site命令。
    // command：命令的内容。
    // 返回值：true-成功；false-失败。
    bool site(const string &command);

    // 获取服务器返回信息的最后一条(return a pointer to the last response received)。
    char *response();
};

} // end namespace idc
#endif

```


## **_ftp.cpp**
```c++
/****************************************************************************************/
/*   程序名：_ftp.cpp，此程序是开发框架的ftp客户端工具的类的定义文件。                  */
/*   作者：吴从周 
/****************************************************************************************/

#include "_ftp.h"

namespace idc
{

cftpclient::cftpclient()
{
    m_ftpconn=0;

    initdata();

    FtpInit();

    m_connectfailed=false;
    m_loginfailed=false;
    m_optionfailed=false;
}

cftpclient::~cftpclient()
{
    logout();
}

void cftpclient::initdata()
{
    m_size=0;

    m_mtime.clear();
}

bool cftpclient::login(const string &host,const string &username,const string &password,const int imode)
{
    if (m_ftpconn != 0) { FtpQuit(m_ftpconn); m_ftpconn=0; }

    m_connectfailed=m_loginfailed=m_optionfailed=false;

    if (FtpConnect(host.c_str(),&m_ftpconn) == false)  { m_connectfailed=true; return false; }

    if (FtpLogin(username.c_str(),password.c_str(),m_ftpconn) == false)  { m_loginfailed=true; return false; }

    if (FtpOptions(FTPLIB_CONNMODE,(long)imode,m_ftpconn) == false) { m_optionfailed=true; return false; }

    return true;
}

bool cftpclient::logout()
{
    if (m_ftpconn == 0) return false;

    FtpQuit(m_ftpconn);

    m_ftpconn=0;

    return true;
}

bool cftpclient::get(const string &remotefilename,const string &localfilename,const bool bcheckmtime)
{
    if (m_ftpconn == 0) return false;

    // 创建本地文件目录。
    newdir(localfilename);

    // 生成本地文件的临时文件名。
    string strlocalfilenametmp=localfilename+".tmp";

    // 获取远程服务器的文件的时间。
    if (mtime(remotefilename) == false) return false;

    // 取文件。
    if (FtpGet(strlocalfilenametmp.c_str(),remotefilename.c_str(),FTPLIB_IMAGE,m_ftpconn) == false) return false;
  
    // 判断文件下载前和下载后的时间，如果时间不同，表示在文件传输的过程中已发生了变化，返回失败。
    if (bcheckmtime==true)
    {
        string strmtime=m_mtime;

        if (mtime(remotefilename) == false) return false;

        if (m_mtime!=strmtime) return false;
    }

    // 重置文件时间。
    setmtime(strlocalfilenametmp,m_mtime);

    // 改为正式的文件。
    if (rename(strlocalfilenametmp.c_str(),localfilename.c_str()) != 0) return false; 

    // 获取文件的大小。
    m_size=filesize(localfilename);

    return true;
}

bool cftpclient::mtime(const string &remotefilename)
{
    if (m_ftpconn == 0) return false;
  
    m_mtime.clear();
  
    string strmtime;
    strmtime.resize(14);

    if (FtpModDate(remotefilename.c_str(),&strmtime[0],14,m_ftpconn) == false) return false;

    // 把UTC时间转换为本地时间。
    addtime(strmtime,m_mtime,0+8*60*60,"yyyymmddhh24miss");

    return true;
}

bool cftpclient::size(const string &remotefilename)
{
    if (m_ftpconn == 0) return false;

    m_size=0;
  
    if (FtpSize(remotefilename.c_str(),&m_size,FTPLIB_IMAGE,m_ftpconn) == false) return false;

    return true;
}

bool cftpclient::chdir(const string &remotedir)
{
    if (m_ftpconn == 0) return false;
  
    if (FtpChdir(remotedir.c_str(),m_ftpconn) == false) return false;

    return true;
}

bool cftpclient::mkdir(const string &remotedir)
{
    if (m_ftpconn == 0) return false;
  
    if (FtpMkdir(remotedir.c_str(),m_ftpconn) == false) return false;

    return true;
}

bool cftpclient::rmdir(const string &remotedir)
{
    if (m_ftpconn == 0) return false;
  
    if (FtpRmdir(remotedir.c_str(),m_ftpconn) == false) return false;

    return true;
}

bool cftpclient::nlist(const string &remotedir,const string &listfilename)
{
    if (m_ftpconn == 0) return false;

    newdir(listfilename.c_str()); // 创建本地list文件目录
  
    if (FtpNlst(listfilename.c_str(),remotedir.c_str(),m_ftpconn) == false) return false;

    return true;
}

bool cftpclient::put(const string &localfilename,const string &remotefilename,const bool bchecksize)
{
    if (m_ftpconn == 0) return false;

    // 生成服务器文件的临时文件名。
    string strremotefilenametmp=remotefilename+".tmp";

    string filetime1,filetime2;
    filemtime(localfilename,filetime1);   // 获取上传文件之前的时间。

    // 发送文件。
    if (FtpPut(localfilename.c_str(),strremotefilenametmp.c_str(),FTPLIB_IMAGE,m_ftpconn) == false) return false;

    filemtime(localfilename,filetime2);   // 获取上传文件之后的时间。

    // 如果文件上传前后的时间不一致，说明本地有修改文件，放弃本次上传。
    if (filetime1!=filetime2) { ftpdelete(strremotefilenametmp); return false; }
    
    // 重命名文件。
    if (FtpRename(strremotefilenametmp.c_str(),remotefilename.c_str(),m_ftpconn) == false) return false;

    // 判断已上传的文件的大小与本地文件是否相同，确保上传成功。
    // 一般来说，不会出现文件大小不一致的情况，如果有，应该是服务器方的原因，不太好处理。
    if (bchecksize==true)
    {
        if (size(remotefilename) == false) return false;

        if (m_size != filesize(localfilename)) { ftpdelete(remotefilename); return false; }
    }

    return true;
}

bool cftpclient::ftpdelete(const string &remotefilename)
{
    if (m_ftpconn == 0) return false;

    if (FtpDelete(remotefilename.c_str(),m_ftpconn) == false) return false;
  
    return true;
}

bool cftpclient::ftprename(const string &srcremotefilename,const string &dstremotefilename)
{
    if (m_ftpconn == 0) return false;

    if (FtpRename(srcremotefilename.c_str(),dstremotefilename.c_str(),m_ftpconn) == false) return false;
  
    return true;
}

bool cftpclient::site(const string &command)
{
    if (m_ftpconn == 0) return false;
  
    if (FtpSite(command.c_str(),m_ftpconn) == false) return false;

    return true;
}

char *cftpclient::response()
{
    if (m_ftpconn == 0) return 0;

    return FtpLastResponse(m_ftpconn);
}

} // end namespace idc

```



