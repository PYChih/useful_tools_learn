P=glist
OBJECTS=
CFLAGS =`pkg-config --cflags glib-2.0` -g -Wall -O3
LDLIBS=`pkg-config --libs   glib-2.0`
CC=gcc

$(P): $(OBJECTS)

.PHONY: clean
clean:
	rm -f $(P)