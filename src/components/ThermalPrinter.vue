<template>
  <div>
    <div class="status">
      <span :class="printerStatusClass">{{ printerStatus }}</span>
    </div>
    <div class="device-info">
      <span>{{ deviceName }}</span>
    </div>
    <button @click="connect" class="btnConnect">Connect</button>
    <button @click="print" class="btnPrint">Print</button>
    <button @click="disconnect" class="btnDisconnect">Disconnect</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      printer: null,
      writer: null,
      printerStatus: "OFF",
      printerStatusClass: "badge bg-danger",
      deviceName: "",
    };
  },
  mounted() {
    if (!("serial" in navigator)) {
      alert("Browser Anda tidak mendukung Web Serial API. Gunakan Google Chrome versi terbaru.");
    }
  },
  methods: {
    async connect() {
      if (this.printer && this.writer) {
        alert("Printer sudah terhubung!");
        return;
      }

      if (!("serial" in navigator)) {
        alert("Browser Anda tidak mendukung Web Serial API.");
        return;
      }

      try {
        const port = await navigator.serial.requestPort();
        this.printer = port;
        this.deviceName = `Menghubungkan...`;
        await this.printer.open({ baudRate: 9600 });
        this.writer = this.printer.writable.getWriter();
        this.printerStatus = "CONNECTED";
        this.printerStatusClass = "badge bg-primary";
        this.deviceName = `Tersambung ke printer`;
      } catch (error) {
        console.error("Error saat menghubungkan ke printer:", error);
        alert("Gagal menghubungkan ke printer. Periksa perangkat Anda.");
      }
    },

    async disconnect() {
      if (!this.printer) {
        alert("Printer belum terhubung.");
        return;
      }

      try {
        if (this.writer) {
          await this.writer.releaseLock();
        }
        await this.printer.close();
        this.printerStatus = "OFF";
        this.printerStatusClass = "badge bg-danger";
        this.deviceName = "";
        this.writer = null;
        this.printer = null;
      } catch (error) {
        console.error("Error saat memutuskan koneksi printer:", error);
        alert("Gagal memutuskan koneksi dengan printer.");
      }
    },

    async print() {
      if (!this.printer || !this.writer) {
        alert("Pastikan printer sudah terhubung.");
        return;
      }

      const printable = {
        Align: {
          center: (text) => '\x1B' + '\x61' + '\x31' + text,
          left: (text) => '\x1B' + '\x61' + '\x00' + text,
          right: (text) => '\x1B' + '\x61' + '\x02' + text,
          reset: () => '\x1B' + '\x61' + '\x31' + '\x1D' + '\x21' + '\x00' + '\n'.repeat(2) + '\r'
        },
        Keyboard: {
          enter: (count) => '\n'.repeat(count) + '\r',
        },
        Font: {
          normal: (text) => '\x1D' + '\x21' + '\x00' + text,
          large: (text) => '\x1D' + '\x21' + '\x11' + text,
        },
        Misc: {
          centerLine: (count) => '\x1B' + '\x61' + '\x31' + '-'.repeat(count)
        }
      }

      const texts = [
        printable.Align.reset(),
        printable.Align.center(printable.Font.large('PT. Ionbit Cafe')),
        printable.Keyboard.enter(1),
        printable.Misc.centerLine(10),
        printable.Keyboard.enter(2),
        printable.Align.left(printable.Font.normal('No. 1')),
        printable.Keyboard.enter(1),
        printable.Align.left(printable.Font.normal('Kasir: Rafi')),
        printable.Keyboard.enter(2),
        printable.Align.left(printable.Font.normal('Short Black (Espresso)')),
        printable.Keyboard.enter(1),
        printable.Align.left(printable.Font.normal('Qty 1 x 22.000 @ 22.000')),
        printable.Keyboard.enter(1),
        printable.Align.left(printable.Font.normal('Iced Long Black')),
        printable.Keyboard.enter(1),
        printable.Align.left(printable.Font.normal('Qty 2 x 25.000 @ 50.000')),
        printable.Keyboard.enter(2),
        printable.Align.left(printable.Font.normal('PPN\t: 12% ')),
        printable.Keyboard.enter(1),
        printable.Align.left(printable.Font.normal('Total\t: Rp. 1.179.920 ')),
        printable.Keyboard.enter(2),
        printable.Align.center(printable.Font.normal("mksh bro")),
        printable.Align.reset(),
        printable.Keyboard.enter(2),
      ]

      this.deviceName = "Sedang mencetak...";
      try {
        for (let text of texts) {
          await this.printText(text);
        }
        alert("Cetak selesai!");
      } catch (error) {
        console.error("Error saat mencetak:", error);
        alert("Gagal mencetak.");
      }
    },

    async printText(text) {
      try {
        const encoder = new TextEncoder();
        const encodedText = encoder.encode(text);
        await this.writer.write(encodedText);
      } catch (error) {
        console.error("Error menulis ke printer:", error);
        alert("Gagal mencetak teks.");
      }
    },
  },
};
</script>

<style scoped>
.status {
  margin-bottom: 10px;
}

.device-info {
  margin-bottom: 10px;
}

.btnConnect,
.btnPrint,
.btnDisconnect {
  margin: 5px;
}
</style>
