6 clone the right repository! not the ...plutus one, it needs to be the ...plutus-apps repo:
```console
git clone https://github.com/input-output-hk/plutus-apps
```
7 goto folder
```console
cd plutus-apps/
```
8 no checkout of commits needed, just use the master branch for now

9 build the server
```console
nix-build -A plutus-playground.server
```
IMPORTANT: if there occurs a segfault, just build it again, it will work.

10 build the client
```console
nix-build -A plutus-playground.client
```
same here, if there are errors during build, just call the command again.

11 start nix shell (takes some time)
```console
nix-shell
```
12 goto server dir and start server (yes, with the GC thing in front)
```console
cd plutus-playground-server
GC_DONT_GC=1 plutus-playground-server
```
sometimes the server will not start at first try. try again, second start should work!

13 wait until server is started! you will see something like this
```console
plutus-playground-server: for development use only
[Info] Running: (Nothing,Webserver {_port = 8080, _maxInterpretationTime = 80s})
Initializing Context
Initializing Context
Warning: GITHUB_CLIENT_ID not set
Warning: GITHUB_CLIENT_SECRET not set
Warning: JWT_SIGNATURE not set
Interpreter ready
```
14 when server is started you can open a second nix-shell in another terminal
```console
nix-shell
```
15 in the second terminal with nix-shell run
```console
sudo npm install -g npm
```
16 goto client dir and start client (yes, with the GC thing in front)
```console
cd plutus-playground-client
GC_DONT_GC=1 npm run start
```
17 wait until client has started and you see something like this
```console
webpack compiled with 1 warning
ℹ ｢wdm｣: Compiled with warnings.
```
will take some time. same here as above: if there are errors be sure to try at least a second build!

if the client takes to long it could be that it runs in timeout which is default 80 seconds.

to prevent this and change the timeout to 150 seconds eg, you can start the server with an option like this:
```console
GC_DONT_GC=1 plutus-playground-server -i 150s
```


both should work now, client with https cert security warning in your browser: https://localhost:8009/

wanna fix the certificate error in your browser? in plutus-playground-client folder do
```console
nano webpack.config.js
```
and change the "https" flag from "true" to "false".

thats it, but not needed. the url from now on would be http://localhost:8009

