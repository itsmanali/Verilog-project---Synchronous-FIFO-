module sync_fifo #(
    parameter DATA_WIDTH = 8,           // Width of each data word
    parameter FIFO_DEPTH = 8,           // Number of storage locations
    parameter ADDR_WIDTH = $clog2(FIFO_DEPTH)  // Address bits
)(
    input  wire                  clk,   // Clock (rising-edge active)
    input  wire                  rst_n, // Active-low synchronous reset
    input  wire                  wr_en, // Write enable
    input  wire                  rd_en, // Read enable
    input  wire [DATA_WIDTH-1:0] din,   // Write data

    output reg  [DATA_WIDTH-1:0] dout,  // Read data
    output wire                  full,  // FIFO full flag
    output wire                  empty, // FIFO empty flag
    output wire [ADDR_WIDTH:0]   count  // Number of valid entries
);

    // ── Internal storage ──────────────────────────────────────
    reg [DATA_WIDTH-1:0] mem [0:FIFO_DEPTH-1];

    // ── Pointers (one extra bit for full/empty discrimination) ─
    reg [ADDR_WIDTH:0] wr_ptr; // write pointer
    reg [ADDR_WIDTH:0] rd_ptr; // read  pointer

    // ── Status flags ──────────────────────────────────────────
    assign count = wr_ptr - rd_ptr;
    assign full  = (count == FIFO_DEPTH);
    assign empty = (count == 0);

    // ── Write logic ───────────────────────────────────────────
    always @(posedge clk) begin
        if (!rst_n) begin
            wr_ptr <= 0;
        end else if (wr_en && !full) begin
            mem[wr_ptr[ADDR_WIDTH-1:0]] <= din;
            wr_ptr <= wr_ptr + 1;
        end
    end

    // ── Read logic (registered output) ────────────────────────
    always @(posedge clk) begin
        if (!rst_n) begin
            rd_ptr <= 0;
            dout   <= 0;
        end else if (rd_en && !empty) begin
            dout   <= mem[rd_ptr[ADDR_WIDTH-1:0]];
            rd_ptr <= rd_ptr + 1;
        end
    end

endmodule
