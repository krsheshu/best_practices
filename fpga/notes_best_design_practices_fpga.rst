==================
Reset Synchization
==================


| The best design practice is to use a 2FF Synronizer for aynchronous reset which can get rid of recovery issues:
| Ref: Asynchronous & Synchronous ResetDesign Techniques - Part Deux- Clifford E. Cummings. Boston 2003

Veriog:

.. code-block:: verilog

  module async_resetFFstyle2 (
    output reg rst_n,
    input      clk, asyncrst_n);

    reg        rff1;

    always @(posedge clk or negedge asyncrst_n)
      if (!asyncrst_n) {rst_n,rff1} <= 2'b0;
      else             {rst_n,rff1} <= {rff1,1'b1};

  endmodule

VHDL:

.. code-block:: vhdl

  library ieee;
  use ieee.std_logic_1164.all;

  entity asyncresetFFstyle is
      port (
        clk        : in  std_logic;
        asyncrst_n : in  std_logic;
        rst_n      : out std_logic);
  end asyncresetFFstyle;

  architecture rtl of asyncresetFFstyle is
    signal rff1 : std_logic;
  begin

    process (clk, asyncrst_n)
    begin
      if (asyncrst_n = '0') then
        rff1  <= '0';
        rst_n <= '0';
      elsif (clk'event and clk = '1') then
        rff1  <= '1';
        rst_n <= rff1;
      end if;
    end process;

  end rtl;

====================================
Test Benches - Reset Race Conditions
====================================

| Time-0 reset deassertion can cause race conditions in simulations. Use the best practice given below to avoid it.
| Ref: Asynchronous & Synchronous ResetDesign Techniques - Part Deux- Clifford E. Cummings. Boston 2003

.. code-block:: verilog

  initial begin     // clock oscillator
    clk <= 0;       // time 0 nonblocking assignment
    forever #(`CYCLE/2) clk = ~clk;
    end

  initial begin
    rst_n <= 0;             // time 0 nonblocking assignment
    @(posedge clk);         // Wait to get past time 0
    @(negedge clk) rst = 1; // rst_n low for one clock cycle
    ...
  end
