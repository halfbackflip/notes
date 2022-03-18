# Writing a simple fuzzer

## Definition
- A fuzzer provides invalid and unexpcted data into the inputs of a program
- This tests the app for buffer overflows, invalid format strings, SQL injection, etc.
- Metasploit provides a complete set of libraries for protocols and data manipulation
- Allows for quick developments of a simple fuzzer

## Metasploit Rex Library
- Module which provides handy methods for handling text
- Random string generation tool for fuzzing
- Example TFTP Fuzzer Tool:
```
#Metasploit

require 'msf/core'

class Metasploit3  '3Com TFTP Fuzzer',
                        'Version'        => '$Revision: 1 $',
                        'Description'    => '3Com TFTP Fuzzer Passes Overly Long Transport Mode String',
                        'Author'         => 'Your name here',
                        'License'        => MSF_LICENSE
                )
                register_options( [
                Opt::RPORT(69)
                ], self.class)
        end

        def run_host(ip)
                # Create an unbound UDP socket
                udp_sock = Rex::Socket::Udp.create(
                        'Context'   =>
                                {
                                        'Msf'        => framework,
                                        'MsfExploit' => self,
                                }
                )
                count = 10  # Set an initial count
                while count < 2000  # While the count is under 2000 run
                        evil = "A" * count  # Set a number of "A"s equal to count
                        pkt = "\x00\x02" + "\x41" + "\x00" + evil + "\x00"  # Define the payload
                        udp_sock.sendto(pkt, ip, datastore['RPORT'])  # Send the packet
                        print_status("Sending: #{evil}")  # Status update
                        resp = udp_sock.get(1)  # Capture the response
                        count += 10  # Increase count by 10, and loop
                end
        end
end
```

## Writing a TFTP Fuzzer Tool
- Metasploit allows for easy creation of new tools by re-using code
- Modify an existing module to create your own module
- Example IMAP Fuzzer Tool
```
##
# This file is part of the Metasploit Framework and may be subject to 
# redistribution and commercial restrictions. Please see the Metasploit
# Framework web site for more information on licensing and terms of use.
# http://metasploit.com/framework/
##


require 'msf/core'


class Metasploit3 > Msf::Auxiliary

    include Msf::Exploit::Remote::Imap
    include Msf::Auxiliary::Dos

    def initialize
        super(
            'Name'           => 'Simple IMAP Fuzzer',
            'Description'    => %q{
                                An example of how to build a simple IMAP fuzzer.
                                Account IMAP credentials are required in this fuzzer.
                        },
            'Author'         => [ 'ryujin' ],
            'License'        => MSF_LICENSE,
            'Version'        => '$Revision: 1 $'
        )
    end

    def fuzz_str()
        return Rex::Text.rand_text_alphanumeric(rand(1024))
    end

    def run()
        srand(0)
        while (true)
            connected = connect_login()
            if not connected
                print_status("Host is not responding - this is G00D ;)")
                break
            end
            print_status("Generating fuzzed data...")
            fuzzed = fuzz_str()
            print_status("Sending fuzzed data, buffer length = %d" % fuzzed.length)
            req = '0002 LIST () "/' + fuzzed + '" "PWNED"' + "\r\n"
            print_status(req)
            res = raw_send_recv(req)
                if !res.nil?
            print_status(res)
                else
                    print_status("Server crashed, no response")
                    break
                end
            disconnect()
        end
    end
end
```