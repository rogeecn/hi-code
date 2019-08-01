---
title: VC++获取网页的内容并转码成UNICODE
date: 2018-01-08 00:00:00
tags: ["VC++"]
abbrlink: visual-cpp-get-webpage-content-and-convert-to-unicode
img: ""
comments: false
---

```cpp
CString UTF8ToUnicode(LPCSTR UTF8)
{
  DWORD dwUnicodeLen;        //转换后Unicode的长度
  TCHAR *pwText;            //保存Unicode的指针
  CString strUnicode;        //返回值
//  LPWSTR pwText;
  //获得转换后的长度，并分配内存
  dwUnicodeLen = MultiByteToWideChar(CP_UTF8,0,UTF8,-1,NULL,0);
  pwText = new TCHAR[dwUnicodeLen];
  if (!pwText)
  {
    return strUnicode;
  }
 
  //转为Unicode
  MultiByteToWideChar(CP_UTF8,0,UTF8,-1,pwText,dwUnicodeLen);
 
  //转为CString
  strUnicode.Format(_T("%s"),pwText);
 
  //清除内存
  //delete []pwText;
 
  //返回转换好的Unicode字串
  return strUnicode;
}
void CdmtestDlg::OnBnClickedButton3()
{
  /*
  CString m_ReturnStr=_T("");//要返回的物理地址
  CString str_addr;
  str_addr = _T("http://www.qoophp.com/index.php");
  //构造访问的地址
  CInternetSession mySession(NULL,0);

  //构造一个新的会话
  CHttpFile* myHttpFile=NULL;
  CString strline;
  myHttpFile=(CHttpFile*)mySession.OpenURL((LPCTSTR)str_addr);//打开网址
  if(myHttpFile!=NULL)
  {
    AfxMessageBox(_T("OpenURL ERROR!"));
    return;
    while(myHttpFile->ReadString(strline))//读取返回的内容
    {
      m_ReturnStr+=strline;
    }
    //m_ReturnStr.Delete(0,m_ReturnStr.Find("来自")+6);
    //CString str=m_ReturnStr.Left(m_ReturnStr.Find("&amp;nbsp"));
    myHttpFile->Close();
    mySession.Close();
  }*/
 
  CString url;
  url = _T("http://www.qoophp.com/index.php");
  //char* headers="Accept:*/*\r\nAccept-Language:zh-cn\r\nUser-Agent:VCTestClient\r\n";
  CInternetSession Sess;
  CHttpFile* cFile = (CHttpFile*)Sess.OpenURL(url,1,INTERNET_FLAG_TRANSFER_ASCII||INTERNET_FLAG_RELOAD);
  DWORD dwStatusCode;
  cFile->QueryInfoStatusCode(dwStatusCode);
  CString szData,szAllData;
  if(dwStatusCode == HTTP_STATUS_OK)
  {
    while(cFile->ReadString(szData))
    {
      szAllData += szData;
      szAllData += "\r\n";
    }
    cFile->Close();
    Sess.Close();
    //CString name = GetFileName(url,TRUE);
    //CFile file(name, CFile::modeCreate | CFile::modeWrite);
    //file.Write(szAllData,szAllData.GetLength());
    //file.Close();
  }
  else
  {
    AfxMessageBox(_T("请求失败。。。。")); 
  }

  AfxMessageBox(szAllData);
  SetDlgItemText(IDC_EDIT1, (LPCTSTR)UTF8ToUnicode((LPCSTR)szAllData.GetBuffer()));
}
```
