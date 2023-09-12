# Qt-Google-OAuth

Sample application to showcase how to use Qt C++ to authenticate Google apps. I have referred part of the code from [Stackoverflow Question](https://stackoverflow.com/questions/48453550/how-to-create-a-login-page-using-qt-oauth) answered by [Xplatforms](https://stackoverflow.com/users/1047746/xplatforms). I will keep the repo updated as Qt and Google introduces more changes.

Feel free to improve the code and post your issues

## Files

#### GoogleOAuth.pro

Remember to add `networkauth` in the list of Qt module dependencies
```sh
QT += networkauth
```

#### googleauth.h & googleauth.cpp

Please read the comments in the files. I have tried to explain all the details of the code.
Remember to edit the `MY_CLIENT_SECRET`, `MY_CLIENT_ID` & `port_number` (in my case, it is 5476) in googleauth.cpp

```sh
this->google->setClientIdentifier("MY_CLIENT_ID");
this->google->setClientIdentifierSharedKey("MY_SECRET_KEY");
auto replyHandler = new QOAuthHttpServerReplyHandler(5476, this);
```

#### main.cpp

Initialized the googleauth object

#### main.qml

Contains the button which calls the `grant()` method of Qt and prompts Google auth on the browser

#### Others

-   [Sample Google JSON File](client_secret_452291785459-1ldhc2q2ahqfl7sv0mh1veuov740bj5f.apps.googleusercontent.com.json)

-   [Google console screenshot](GoogleConsole.png): Remember to put http://127.0.0.1:your_port instead of http://localhost:your_port else this will result in `redirect_uri mismatch error`


#方法2
-	通过url
	webengine 加载   https://accounts.google.com/o/oauth2/v2/auth?client_id=573535045251-jdud2c4gcec30h5srfh8m6kqf80elngo.apps.googleusercontent.com&redirect_uri=http://127.0.0.1:5476/&scope=email&response_type=code
	获取code
	从而获取
	    grant_type = "authorization_code";
    client_id = q.queryItemValue("client_id");
    client_secret = "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";
    redirect_uri = q.queryItemValue("redirect_uri");
    QJsonObject bodyobj;
    bodyobj.insert("grant_type", grant_type);
    bodyobj.insert("client_id", client_id);
    bodyobj.insert("client_secret", client_secret);
    bodyobj.insert("redirect_uri", redirect_uri);
    bodyobj.insert("code", code);
    QJsonDocument doc(bodyobj);
    body = doc.toJson();

//    sig_tripartite_login("google", "", code);
    HttpHelper *tokenreq = new HttpHelper;
    connect(tokenreq, &HttpHelper::signalPostFinished, [code, this](int nCode, QString req){
        QJsonParseError parseError;
        QByteArray json = req.toUtf8();
        QJsonDocument doc = QJsonDocument::fromJson(req.toUtf8(), &parseError);
        if (parseError.error != QJsonParseError::NoError)
        {
            LOG(LERROR) << "Parse error:" << parseError.errorString().toStdString() << ", Response: " << req.toStdString();
            return ;
        }
        qDebug()<<"req==="<<req;
        QJsonObject obj = doc.object();
        QString access_token = obj.value("access_token").toString();
        int expires_in = obj.value("expires_in").toInt(-1);
        QString refresh_token = obj.value("refresh_token").toString();
        QString scope = obj.value("scope").toString();
        QString id_token = obj.value("id_token").toString();
        QString token_type = obj.value("token_type").toString();

        sig_tripartite_login("google", id_token, code);
        HttpHelper *preq = (HttpHelper*)(sender());
        preq->deleteLater();
    });

    tokenreq->PostAsync("https://oauth2.googleapis.com/token", body);
	
	https://juejin.cn/post/7252584767593627707
	////////////////////////////
	