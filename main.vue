<template>
  <div>
    <div class="header">
      <input
        type="file"
        name="ggg"
        @change="open"
      >
    </div>
    <div class="browser">
      <div
        v-for="(r,i) in records"
        :key="i"
        class="record"
        :style="{'background-color': r.info.b, 'color': r.info.f}"
      >

        <div class="note">
          <div class="title">
            <div class="icon"><i :class="'fas fa-'+icon[r.i]" /></div>{{ r.title }}
          </div>
          <div
            v-for="(n, j) in r.N"
            :key="'note' + j"
            class="section"
          >{{ n }}</div>
        </div>
        <div class="zone">
          {{ r.Z }}
          <div
            v-if="r.M"
            class="coor"
          >{{ r.M.join(', ') }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
@import "node_modules/@fortawesome/fontawesome-free/css/all.css";

html {
  font-family: "Source Sans Pro", -apple-system, BlinkMacSystemFont, "Segoe UI",
    Roboto, "Helvetica Neue", Arial, sans-serif;
  font-size: 12px;
  word-spacing: 1px;
  -ms-text-size-adjust: 100%;
  -webkit-text-size-adjust: 100%;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  -webkit-user-select: none;
  -webkit-user-drag: none;
  box-sizing: border-box;
  overflow: auto;
}

a {
  color: #686;
}

*,
*:before,
*:after {
  box-sizing: border-box;
  margin: 0;
}

.header {
  height: 80px;
  width: 100%;
  padding: 20px;
}

.record {
  width: 500px;
  padding: 15px;
  margin: 2px;
  border-radius: 5px;
}

.title {
  font-size: 20px;
  margin-bottom: 5px;
}

.icon {
  display: inline-block;
  margin-right: 10px;
}

.note {
  width: calc(100% - 80px);
  display: inline-block;
}

.section {
  margin: 10px;
}

.zone {
  width: 79px;
  display: inline-block;
  vertical-align: top;
  font-size: 16px;
  text-align: right;
}

.coor {
  font-size: 10px;
  margin-top: 20px;
}
</style>

<script>
import md5 from "md5";
import fc from "false-color";

export default {
  data() {
    return {
      records: [],
      icon: {
        A: "exclamation",
        b: "anchor",
        B: "coins",
        C: "check-circle",
        f: "crow",
        F: "helicopter",
        h: "hand-holding-heart",
        H: "heart",
        K: "skull-crossbones",
        l: "hand-holding",
        L: "level-up-alt",
        N: "clipboard",
        P: "exchange-alt",
        r: "hammer",
        R: "running",
        t: "question-circle",
        T: "question",
        U: "hand-point-right",
        ";": "comments",
        $: "gem"
      }
    };
  },

  mounted() {
    // let vm = this;
  },

  methods: {
    log: (...x) => console.log(...x),
    color(s) {
      if (!s) s = "555";
      // this.log("converting", s, "to", parseInt(md5(s).slice(0, 2), 16));
      return fc.range(0, 255).convert(parseInt(md5(s).slice(0, 2), 16));
    },
    parse(s) {
      let vm = this;
      let rec = [];
      let nop = /ACH|BUFF|BUILDING|CC|CHAT|CN|CS|LEAD|LVL|NA|NC|O|P|PET|RANK|RECIPE|REP|S|S!US|SPELL|TZ|US|V/;
      let pre = {
        QID: s => parseFloat(s),
        M: s => s.split(",").map(x => parseFloat(x))
      };

      let n = 0;
      s.split("\n").forEach(l => {
        n++;
        if (l.match(/[a-zA-Z] *\|/)) {
          // vm.log("line:", l);

          let r = {};
          let x;
          l.split("|").forEach((seg, i) => {
            if (!i) {
              r.i = seg[0];
              r.title = seg.slice(2);
              return;
            }

            if (x) {
              r[x] = x in pre ? pre[x](seg) : seg;
              x = null;
            } else if (seg.match(nop)) r[seg] = 1;
            else x = seg;
          });

          if (r.N) r.N = r.N.split("\\n");
          r.info = { l };

          let c = vm.color(r.Z);
          r.info.b = c.toString("hex");
          r.info.f = c.fg().toString("hex");
          // vm.log("r:", r);
          rec.push(r);
        } else {
          vm.log("not match", l);
        }
      });

      vm.log(rec.length, "records", n, "lines");
      vm.records = rec;
    },
    open(e) {
      let vm = this;

      vm.log("open", e.target.files);

      for (let j = 0; j < e.target.files.length; j++) {
        let file = e.target.files[j];

        // browser
        // let blob = file.slice(offs, offs + 0x10000)
        let r = new FileReader();
        r.onload = e => {
          if (e.target.error) {
            vm.log("read file error: " + e.target.error);
            return;
          }

          let res = "";
          let dec = new TextDecoder();
          res = dec.decode(e.target.result);

          vm.log("parsing", res.length / 1000, "KB text");
          vm.parse(res);
        };

        r.readAsArrayBuffer(file);
      }

      e.target.value = "";
    }
  }
};
</script>
