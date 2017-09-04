
# About

m_net was a cross platform network library, provide a simple and
efficient interface for covenient use.

Support MacOS/Linux/Windows, using kqueue/epoll/select underlying.




# Features

- simple API in C++ wrapper
- with TCP/UDP support
- nonblocking & event driven interface
- using kqueue/epoll/select in MacOS/Linux/Windows




# Usage & Example

It's very convenience for use, an echo server example:

```cpp
// subclass Chann to handle event
class CntChann : public Chann {
public:
   CntChann(Chann *c) { channUpdate(c, NULL); }

   // implement virtual defaultEventHandler
   void defaultEventHandler(Chann *accept, chann_event_t event) {

      if (event == CHANN_EVENT_RECV) {
         memset(m_buf, 0, 256);
         int ret = channRecv(m_buf, 256);
         channSend(m_buf, ret);
      }
      if (event == CHANN_EVENT_DISCONNECT) {
         delete this;           // release chann
      }
   }
   char m_buf[256];
};

int main(int argc, char *argv[]) {
   if (argc < 2) {
      cout << argv[0] << ": 'svr_ip:port'" << endl;
   } else {
      cout << "svr start listen: " << argv[1] << endl;

      Chann echoSvr("tcp");
      echoSvr.channListen(argv[1]);
   
      echoSvr.setEventHandler([](Chann *self, Chann *accept, chann_event_t event) {
            if (event == CHANN_EVENT_ACCEPT) {
               CntChann *cnt = new CntChann(accept);
               cnt->channSend((void*)("Welcome to echoServ"), 20);
               delete accept;
            }
         });
      ChannDispatcher::startEventLoop();
   }
   return 0;
}
```

the nested code need Closure support, with environment:

- Apple LLVM version 8.1.0 (clang-802.0.42)
- g++ (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3


and the client side example:

```cpp
static void
_getUserInputThenSend(Chann *self) {
   string input; cin >> input;
   if (input.length() > 0) {
      self->channSend((void*)input.c_str(), input.length());
   }
}

// external event handler
static void
_cntEventHandler(Chann *self, Chann *accept, chann_event_t event) {
   if (event==CHANN_EVENT_CONNECTED || event==CHANN_EVENT_RECV) {
      char buf[256] = {0};
      int ret = self->channRecv(buf, 256);
      if (ret > 0) {
         cout << buf << endl;
         _getUserInputThenSend(self);
      }
   }
   if (event == CHANN_EVENT_DISCONNECT) {
      ChannDispatcher::stopEventLoop();
   }
}

int
main(int argc, char *argv[]) {

   if (argc < 2) {
      cout << argv[0] << ": 'cnt_ip:port'" << endl;
   } else {
      cout << "cnt try connect " << argv[1] << "..." << endl;

      Chann cnt("tcp");
      cnt.setEventHandler(_cntEventHandler);
      cnt.channConnect(argv[1]);

      ChannDispatcher::startEventLoop();

      cnt.channDisconnect();
   }

   return 0;
}
```

In the other hand, the C interface with more flexible options.

Feel free to drop plat_net.[ch] to your project, more example and test
case can be found in relative dir.

A more complex demo, the [m_tunnel](https://github.com/lalawue/m_tunnel)
project also using this library to provide a secure socks5 interface tcp
connection between local <-> remote side.




# Thanks

Thanks NTP source code from https://github.com/edma2/ntp, author: Eugene Ma (edma2)