P=gsl_erf
OBJECTS=
CFLAGS = -I$(HOME)/root/include -g -Wall -O3
LDLIBS=-L$(HOME)/root/lib -Wl,-R$(HOME)/root/lib -lgsl -lgslcblas -lm
CC=c99

$(P): $(OBJECTS)

.PHONY: clean
clean:
	rm -f $(P)