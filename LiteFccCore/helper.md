Send hex data to the Tcp server, and display the response with hex-ASCII table
```
echo -e "\x50\x20\xFA" | netcat 192.168.1.119 1234 | hexdump -C
```