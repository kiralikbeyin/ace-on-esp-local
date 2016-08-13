# ace-on-esp-local
ace files without cdn

ace.js embeded to edit.html.gz and renamed long theme name to duru

---listing directories may delay 20 seconds---

orginal code from here
https://github.com/me-no-dev/ESPAsyncWebServer

you can serve edit.html.gz file and use with ;


```


class SPIFFSEditor: public AsyncWebHandler {
  private:
    String _username = httpAuth.wwwUsername.c_str();
    String _password = httpAuth.wwwPassword.c_str();
    bool _authenticated;
    uint32_t _startTime;
  public:
    bool bak;
    SPIFFSEditor(String username = String(), String password = String()): _username(username), _password(password), _authenticated(false), _startTime(0) {}
    bool canHandle(AsyncWebServerRequest *request) {
      if (request->method() == HTTP_GET && request->url() == "/edit")
        return true;
      else if (request->method() == HTTP_GET && request->url() == "/list")
        return true;
      else if (request->method() == HTTP_GET && !(request->url().endsWith("/")) && request->hasParam("download"))
        return true;
      else if (request->method() == HTTP_POST && request->url() == "/edit")
        return true;
      else if (request->method() == HTTP_DELETE && request->url() == "/edit")
        return true;
      else if (request->method() == HTTP_PUT && request->url() == "/edit")
        return true;

      return false;
    }


    void handleRequest(AsyncWebServerRequest *request) {

      if (_username.length() && _password.length() && !request->authenticate(_username.c_str(), _password.c_str()))
        return request->requestAuthentication();
      if (request->method() == HTTP_GET && request->url() == "/list") {
        if (request->hasParam("dir")) {
          String path = request->getParam("dir")->value();
          Dir dir = SPIFFS.openDir(path);
          path = String();
          String output = "[";
          while (dir.next()) {
            fs::File entry = dir.openFile("r");
            if (output != "[") output += ',';
            bool isDir = false;
            output += "{\"type\":\"";
            output += (isDir) ? "dir" : "file";
            output += "\",\"name\":\"";
            output += String(entry.name()).substring(1);
            output += "\"}";
            entry.close();
          }
          output += "]";
          request->send(200, "text/json", output);
          output = String();
        }

      }
      else  if (request->method() == HTTP_GET && request->url() == "/edit") {


        request->send(SPIFFS, "/edit.html");

      }
      else if (request->method() == HTTP_GET) {
        request->send(SPIFFS, request->url(), String(), true);
      } else if (request->method() == HTTP_DELETE) {
        if (request->hasParam("path", true)) {
          SPIFFS.remove(request->getParam("path", true)->value());
          request->send(200, "", "DELETE: " + request->getParam("path", true)->value());
        } else
          request->send(404);
      } else if (request->method() == HTTP_POST) {
        if (request->hasParam("data", true, true) && SPIFFS.exists(request->getParam("data", true, true)->value()))
          request->send(200, "", "UPLOADED: " + request->getParam("data", true, true)->value());
        else
          request->send(500);
      } else if (request->method() == HTTP_PUT) {
        if (request->hasParam("path", true)) {
          String filename = request->getParam("path", true)->value();
          if (SPIFFS.exists(filename)) {
            request->send(200);
          } else {
            fs::File f = SPIFFS.open(filename, "w");
            if (f) {
              f.write((uint8_t)0x00);
              f.close();
              request->send(200, "", "CREATE: " + filename);
            } else {
              request->send(500);
            }
          }
        } else
          request->send(400);
      }
    }

    void handleUpload(AsyncWebServerRequest *request, String filename, size_t index, uint8_t *data, size_t len, bool final) {
      if (!index) {
        if (!_username.length() || request->authenticate(_username.c_str(), _password.c_str())) {
          _authenticated = true;
          request->_tempFile = SPIFFS.open(filename, "w");
          _startTime = millis();
        }
      }
      if (_authenticated && request->_tempFile) {
        if (len) {
          request->_tempFile.write(data, len);
        }
        if (final) {
          request->_tempFile.close();
          uint32_t uploadTime = millis() - _startTime;
          os_printf("upload: %s, %u B, %u ms\n", filename.c_str(), index + len, uploadTime);
          String yuklendi = "Dosya yüklendi: " + filename + ", " + (index + len) / 1000 + " KB, " + uploadTime / 1000 + " saniye";
          ws.textAll(yuklendi);

        }
      }
    }


};

```
