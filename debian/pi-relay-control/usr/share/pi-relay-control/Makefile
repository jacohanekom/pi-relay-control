CXX      = g++
CXXFLAGS = -Wall -O2 -std=c++17
LDFLAGS  = -llgpio
TARGET   = relay_control
SRCDIR   = src

all: $(TARGET)

$(TARGET): $(SRCDIR)/main.cpp
	$(CXX) $(CXXFLAGS) -o $(TARGET) $(SRCDIR)/main.cpp $(LDFLAGS)

install:
	install -d $(DESTDIR)/usr/bin
	install -m 755 $(TARGET) $(DESTDIR)/usr/bin/
	install -d $(DESTDIR)/usr/share/pi-relay-control/src
	install -m 644 $(SRCDIR)/main.cpp $(DESTDIR)/usr/share/pi-relay-control/src/
	install -m 644 Makefile $(DESTDIR)/usr/share/pi-relay-control/
	install -d $(DESTDIR)/etc
	install -m 644 debian/pi-relay-control.conf $(DESTDIR)/etc/pi-relay-control.conf

clean:
	rm -f $(TARGET)
