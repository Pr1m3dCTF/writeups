# 1 - Challenge code and description

```
As you venture further into the depths of the tomb, your communication with your team becomes increasingly disrupted by noise.
Despite their attempts to encode the data packets, the errors persist and prove to be a formidable obstacle.
Fortunately, you have the exact Verilog module used in both ends of the communication.
Will you be able to discover a solution to overcome the communication disruptions and proceed with your mission?
```

```sv
module encoder(
    input [3:0] data_in,
    output [6:0] ham_out
    );
 
    wire p0, p1, p2;
 
    assign p0 = data_in[3] ^ data_in[2] ^ data_in[0];
    assign p1 = data_in[3] ^ data_in[1] ^ data_in[0];
    assign p2 = data_in[2] ^ data_in[1] ^ data_in[0];
    
    assign ham_out = {p0, p1, data_in[3], p2, data_in[2], data_in[1], data_in[0]};
endmodule

module main;
    wire[3:0] data_in = 5;
    wire[6:0] ham_out;

    encoder en(data_in, ham_out);

    initial begin
        #10;
        $display("%b", ham_out);
    end
endmodule
```

Here we have a verilog code which splits input data into `4 bit arrays` and generate a parity checksum for them with `encoder` (p0,p1,p2)\
So in output data we have `4 bit input data` and parity check `p0,p1,p2` which is for error detection\
According to the challenge description this data is transfer and there are a lot of noice on it.


# 2 - Solution

To solve this lab we should use that parity check bits to ensure if the data was changed or got any noice or not.\
Overall solution is like below:

1. Capture raw binary data
2. Split it into `7 bit` pieces
3. generate a python dictionary foreach block with default value of `False` for each block which indicates we determine all blocks to got errored
3. Check parity for each `7 bit` block with chech_parity function and if it is true update it in dictionary with the value `True`
4. Because each captured data has random noices, Repeat these steps and update each `7 bit block` if it has True parity_check until all blocks have True parity check
5. get `4 bit` raw data from `7 bit` blocks concat to gether and decode it from binary to ascii


Here is the overall process in python
```py
import socket

HOST = '167.172.50.208'
PORT = 30920

def parse(data):
    return data[2] + data[4] + data[5] + data[6]


def check_parity(data_in):
    
    enc = [int(i) for i in data_in]

    p0 = enc[2] ^ enc[4] ^ enc[6]
    p1 = enc[2] ^ enc[5] ^ enc[6]
    p2 = enc[4] ^ enc[5] ^ enc[6]

    return (p0 == enc[0] and p1 == enc[1] and p2 == enc[3])


def check_result(data):
    for d in data:
        if not data[d]:
            return False
        
    return True

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((HOST, PORT))
results = []
result = {}
for i in range(0, 952, 7):
    result[i] = ''
raw = ''

while True:
    raw_enc = sock.recv(2048).decode().strip()[10:]

    size = len(raw_enc)
    encs = [raw_enc[i:i+7] for i in range(0, size, 7)]

    index = 0
    for raw_enc in encs:
        if check_parity(raw_enc):
            result[index] = raw_enc
        index += 7

    if check_result(result):
        raw = ''.join([result[d] for d in result])
        break


index = 0
message = ''
size = len(raw)

for i in range(0, size, 7):
    message += parse(raw[i:i+7])
    index += 1

flag = bytes.fromhex(hex(int(message,2))[2:])
print(flag)
```

If we run the code we get a flag but it is still noised the reason is that this custom parity check is not 100% true for error detection
```bash
python solve.py
b'HTB{hcm_w?thWs0m3\x1fo.a\x11ys15_y\x10u_c4n_3\xf87ract_\x07hs_h4mmIn9_7_t_3>\xe3_fl49y'
```

So we should repeat executing it an getting errored flags and manually fix noised characters\
Here are 10 output each one gives us different data
```
HTB{hmm_w1th_s0m3_ana1ys15_y0u_c4n_3x7ract_7h3_h4mmin9_7_4_3nc_fl49}
HTB{hmo_y1äh_s0m;_ana1ys15_yÐu_C4n_3q7rac_7l³_è0mm)n;_7[4_;nc_fl4yt
HTF{hmm¿w1ta_s0m3ÏanB1's1u_y0u¿ó4n_cx7rdct_7h3_h4mmkk9_7_:_snc_fl49t
HTB{hým_w1th_s0m3_anôys15_y0u_#4n_3xâact_7h3_h4mmi9_7_4_3nf_fl49s
HTBhhm_t1th_s0ý3_ana1ws15_ypu_#´n[6x×rct_7h3[hmmgn9_7_4_3nc_fl:7}
HTBhmm_w1th_s0m3_ala1}#15y0u_c4n_6x7rac$_©H6_l4emin:_§_4_3nc_fl4yý
HtBr(=m_^1th_s0m3_hna1ys15]y0u_cn_3x7raó}_7f3]h4mnin¹g_4_3^c_fl49}
HTAyh=m_w1tøaq0m_!na1ys5_y0u_c4n_3x×rac_7h3_h4omin9\_4_3nj_fl49}
xTB{hmm_w1Dh¿30m3_ana1ép15_y0u_cn_3x7ract_5h=_h:mmin9_74_3þjßfl¤9}
HB{hãm_'1th_ã0m3_aga1ys15\y0u_6n_3x7racD_×m3_h4cmin9ß7_4_3kc_fl49}
```

We can manually extract correct flag characters based on words meanings\
And here is the final correct flag
```
HTB{hmm_w1th_s0m3_ana1ys15_y0u_c4n_3x7ract_7h3_h4mmin9_7_4_3nc_fl49}
```

