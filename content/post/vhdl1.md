---
title: "verilog HDL 기본문법"
date: 2021-11-08T23:39:51+09:00
 - verilog_HDL
tags:
 - verilog_HDL
---

- Inverter

- assign 구문 사용
  - 대부분의 경우 assign을 사용
  - "조건 ? 참일경우 : 거짓일경우" 의 문장을 자주 사용
- always 구문 사용
  always @ (sensitivity_list) begin
  ...
  end

- always : 언제나, @ : (sensitivity_list) 신호가 변할 때
- 논리가 복잡한 경우에 사용
- 이 경우 if, else if, else 구문 또는 case 구문을 주로 사용
- @(sensitivity_list) : 감지신호 목록
  - sensitivity_list에 나열된 신호들 중 하나라도 값이 바뀌면 내부 문장이 실행
  - sensitivity_list에는 출력에 영향을 주는 모든 신호가 나열되어야 함

- assign 구문을 사용한 작성

        wire out;
        assign out = ~in;

        wire out;
        assign out = (in == 0)

        wire out;
        assign out = ~(in == 1)

        wire out;
        assign out = in ? 0 : 1;

        wire out;
        assign out = ~in ? 1 : 0;

- always 구문을 사용한 작성

        reg out;
        always @(in)
        out = ~in;

        reg out;
        always @(in) begin
        if (in) out = 0;
        else out = 1;
        end

        reg out;
        always @(in) begin
        case (in)
            0 : out = 1;
            1 : out = 0;
            default : out = 1'bx;
        endcase
        end

- 인버터 모듈
  - 2개의 입출력 포트(port)
  - 입력신호는 in, 출력 신호는 out

        module inverter (in, out);
        input in;
        output out;

        wire out;
        assign out = ~in;
        endmodule

- OR gate(3-input OR gate)

        wire out;
        assign out = in[2] | in[1] | in[0];

        wire out;
        assign out = |in[2:0];

        wire out;
        assign out = (in[2:0] != 3`b000);

        wire out;
        assign out = ~(in[2:0] == 3`b000);

- NOR gate

        wire out;
        assign out = ~(in[2] | in[1] | in[0]);

        wire out;
        assign out = ~|in[2:0];

        wire out;
        assign out = ~(in[2:0] != 3`b000);

        wire out;
        assign out = in[2:0] == 3`b000;

- 반가산기
  - 2개의 입력을 받아 덧셈을 수행하고 sum과 carry를 발생

        module half_adder (a, b, sum, cout);
        input a, b;
        output sum, cout;
        wire sum, cout;

        assign sum = a ^ b;
        assign cout = a & b;
        endmodule

- 전가산기
  - 3개의 입력을 받아 sum과 carry를 발생
  - 반가산기 2개와 or 게이트를 사용

        module full_adder (a, b, cin, sum, cout);
        input a, b, cin;
        output sum, cout;
        wire sum, cout;

        assign sum = (a ^ b) ^ cin;
        assign cout = ((a ^ b) & cin) | (a & b);
        endmodule

Multiplexer(mux)
- 여러 개의 입력 중 하나만을 선택하여 출력
  - sel == 1 이면 out = a
  - sel == 0 이면 out = b

- assign을 사용

        wire out;
        assign out = (sel == 1) ? a : b;

        wire out;
        assign out = sel ? a : b;

- always를 사용

        reg out;
        always @(sel or a or b)
        out = sel > a : b;

        reg out;
        always @(sel or a or b) begin
        if (sel) out = a;
        else out = b;
        end

        module mux2to1 (a, b, sel, out);
        input a, b, sel;
        output out;
        reg out;
        always @(sel or a or b) begin
            case (sel)
            1`b1 : out = a;
            1`b0 : out = b;
            endcase
        end
        endmodule

-multi-bit mux

        wire [3:0] out;
        assign out[3:0] = sel ? a[3:0] : b[3:0];

        module mux4bit (a, b, sel, out);
        input [3:0] a, b;
        input sel;
        output [3:0] out;

        reg [3:0] out;
        always @(sel or a or b) begin
            case(sel)
            1'b1 : out = a; // a, b, out 이 모두 4비트로
            1'b0 : out = b; // 선언되어 있으므로 비트표시 생략, out[3:0] = a[3:0]
            endcase
        end
        endmodule

- 가산기(adder)
  - 가산기 작성시 '+' 연산자 사용
  - 합성시에 자동으로 속도에 맞는 가산기가 발생됨
  - 4비트 + 4비트 carry와  4비트의 sum

        wire carry_out;
        wire [3:0] sum;
        assign { carry_out, sum[3:0] } = a[3:0] + b[3:0];

- 가감산기(adder and subtractor)
  - add_nsub == 1 : 가산
  - add_nsub == 0 : 감산

        wire carry_out;
        wire [3:0] sum;
        assign {carry_out, sum[3:0]} = add_nsub > a[3:0] + b[3:0] : a[3:0] + ~b[3:0] + 4'b1;

- 디코더
  - 코드화된 신호를 받아 하나의 출력만을 1 또는 0으로 구동

        wire[3:0] y;

        assign y[0] = !a[1] & !a[0];
        assign y[1] = !a[1] & a[0];
        assign y[2] = a[1] & !a[0];
        assign y[3] = a[1] & a[0];

        wire[3:0] y;

        assign y[0] = a[1:0] == 2'b00;
        assign y[1] = a[1:0] == 2'b01;
        assign y[2] = a[1:0] == 2'b10;
        assign y[3] = a[1:0] == 2'b11;

        wire[3:0] y;

        assign y = (a == 2'b00) ? 4'b0001 :
                    (a == 2'b01) ? 4'b0010 :
                    (a == 2'b10) ? 4'b0100 :
                                        4'b1000;

        reg[3:0] y;

        always @(a) begin
        case(a)
            2'b00 : y = 4'b0001;
            2'b01 : y = 4'b0010;
            2'b10 : y = 4'b0100;
            2'b11 : y = 4'b1000;
        endcase
        end

- 인코더
  - 디코더와 반대의 동작으로 개별 신호를 묶어 코드화

        wire b, a, v
        assign b = d3 || (!d3 & d2);
        assign a = d3 || (!d3 & !d2 & d1);
        assign v = !(!d3 & !d2 & !d1 & !d0);

- ROM
  - ROM은 메모리로 분류되지만, 동작관점에서는 조합논리회로임.

- DFF

        module d_ff (clk, d, q);
        input clk;
        input d;
        output q;

        reg q;
        always @(posedge clk)
            q <= d;

        endmodule

- DFF with asychronous reset

        reg q;
        always @(posedge clk or negedge rst) begin
        if (!rst) q <= 0;
        else q <= d;
        end

- DFF with enable

        reg q;
        always @(posedge clk or negedge rst) begin
        if(!rst) q <= 0;
        else if (en) q <= d;
        else q <= q; // needless
        end

- DFF with q and qbar outputs

        reg q, q_bar;
        always @(posedge clk) begin
        q <= d;
        q_bar <= ~d;
        end

        reg q;
        wire q_bar;

        always @(posedge clk)
        q <= d;
        assign q_bar = ~q;
