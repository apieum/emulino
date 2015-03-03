Emulino
=======

_originally taken from https://wiki.ubuntu.com/Emulino_

Emulino is an emulator for the Arduino platform by Greg Hewgill,
originally found on emulino: arduino cpu emulator.

With it, you can build ("Verify") a .pde sketch in Arduino IDE which
uses serial port for communication to say a .hex file, and then you
can invoke the emulino executable with the .hex file as an argument,
and get a local simulation of how an Arduino would behave; useful for
debugging without wasting burn cycles on the Arduino's AVR chip.

## Building

Here is a brief overview of steps to get emulino built.

    sudo apt-get install clang-3.6
    sudo apt-get install git # to check out the source
    sudo apt-get install scons # to build the source

    git clone git://github.com/ghewgill/emulino.git
    cd emulino/
    scons install

The result is an `emulino` executable installed in `/usr/bin`.

### Using Vagrant

You need to have [Vagrant](https://www.vagrantup.com/),
[VirtualBox](https://www.virtualbox.org/) and
[Ansible](http://www.ansible.com/home), but this handles all of
provisioning for you. Simply do

    $> vagrant up
    $> vagrant ssh
    $> cd /vagrant
    $> scons

and you will have an `emulino` executible in `<project-root>/build`

## Usage

For instance, one could use the following simple code, which simply
increases a counter and writes its value on serial, as test.pde in
Arduino IDE:

    int counter = 0;

    void setup()
    {
        // initiate Arduino serial communication
        Serial.begin(115200);
    }

    void loop()
    {
        counter = counter + 1;
        Serial.print("Counter: ");
        Serial.print(counter, DEC);
        Serial.println();
        delay(50);
    }

By clicking on "Verify", the code is built - and during a "Verify" the
.hex file is written to /tmp. In order to simulate the code, one can
simply call emulino with the path to the hex file as argument:


    $> ./emulino /tmp/build523158875647966391.tmp/test.cpp.hex
    emulino: Loading hex image: /tmp/build523158875647966391.tmp/test.cpp.hex
    Counter: 1
    Counter: 2
    Counter: 3
    Counter: 4
    ...

## Serial-'std' in-out

Note that, in principle, emulino uses stdin and stdout of the console
to simulate serial input/output. There is a command switch "-io" which
should allow input/output to/from files (which have fixed names of
name-of-hexfile.hex.in and name-of-hexfile.hex.out), however I cannot
get it to work every time (maybe instead of files, one should use
mknod and named pipes; not sure).

Note however the example below - which reads a character from serial
and echoes it back to serial:


    static unsigned char _c;

    void setup()
    {
        Serial.begin(9600);
    }

    void loop()
    {

      // read in a byte, and send it back immediately
      // send data only when you receive data:
      if (Serial.available() > 0)
      {
          // read the incoming byte:
          _c = Serial.read();

          // say what you got:
          Serial.print(_c);
      }

      // must have delay for emulino to work
      delay(1);
    }

In this example, if the delay statement is missing, then emulino will
never register that a character came in from "serial"; with the delay
statement, a command like session will look like:

    $> stty -icanon ; ./emulino /tmp/buildXXX.tmp/test.cpp.hex
    emulino: Loading hex image: /tmp/buildXXX.tmp/test.cpp.hex
    tteessttiinngg  bbuuttttoonn  pprreessss^C

Note again that "stty -icanon" seems to be needed, in order to set up
the terminal as a character device - so that keypresses are passed on
to "stdin" per individual character basis.¯
