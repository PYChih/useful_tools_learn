P=DisplayImage
OBJECTS=DisplayImage.o
SRCS=DisplayImage.cpp
CFLAGS = `pkg-config --cflags opencv`-g -Wall -O3
LDLIBS= `pkg-config --libs opencv`
CC=g++

$(P): $(OBJECTS)

DisplayImage.o:DisplayImage.cpp
	g++ -c DisplayImage.cpp -o DisplayImage.o

.PHONY: clean
clean:
	rm -f *.o $(P)