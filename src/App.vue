<template>
  <div id="app">
    <h1>请求调页算法模拟</h1>
    <el-button type="primary" @click="main">开始模拟</el-button>
    <el-container style="solid #eee">
      <el-row :gutter="100">
        <el-col :span="8">
          <el-card style="border-radius: 12px">
            <h3>模拟结果</h3>
            <el-table :data="result" stripe style="width: 500px">
              <el-table-column prop="algorithm" label="算法" align="center">
              </el-table-column>
              <el-table-column prop="count" label="缺页次数" align="center">
              </el-table-column>
              <el-table-column prop="ratio" label="缺页率" align="center">
              </el-table-column>
            </el-table>
          </el-card>
        </el-col>
        <el-col :span="16">
          <el-card style="border-radius: 12px">
            <h3>显示内存中的页面</h3>
            <el-row>
              <el-col :span="16">
                <el-select v-model="value" placeholder="请选择">
                  <el-option
                    v-for="item in options"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  >
                  </el-option>
                </el-select>
              </el-col>
              <el-col :span="8">
                <el-button type="primary" @click="getRecord">查看</el-button>
              </el-col>
            </el-row>
            <el-table :data="record" stripe style="width: 800px" height="400px">
              <el-table-column prop="num" label="指令" align="center">
              </el-table-column>
              <el-table-column prop="addr" label="逻辑地址" align="center">
              </el-table-column>
              <el-table-column prop="paddr" label="物理地址" align="center">
              </el-table-column>
              <el-table-column prop="miss" label="是否缺页" align="center">
              </el-table-column>
              <el-table-column prop="mem0" label="内存块0" align="center">
              </el-table-column>
              <el-table-column prop="mem1" label="内存块1" align="center">
              </el-table-column>
              <el-table-column prop="mem2" label="内存块2" align="center">
              </el-table-column>
              <el-table-column prop="mem3" label="内存块3" align="center">
              </el-table-column>
            </el-table>
          </el-card>
        </el-col>
      </el-row>
    </el-container>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      addr: [],
      page: [],
      record: [],
      result: [
        {
          algorithm: "OPT",
          count: 0,
          ratio: 0,
        },
        {
          algorithm: "FIFO",
          count: 0,
          ratio: 0,
        },
        {
          algorithm: "LRU",
          count: 0,
          ratio: 0,
        },
        {
          algorithm: "LFU",
          count: 0,
          ratio: 0,
        },
        {
          algorithm: "Clock",
          count: 0,
          ratio: 0,
        },
      ],
      options: [
        {
          value: "OPT",
          label: "OPT",
        },
        {
          value: "FIFO",
          label: "FIFO",
        },
        {
          value: "LRU",
          label: "LRU",
        },
        {
          value: "LFU",
          label: "LFU",
        },
        {
          value: "Clock",
          label: "Clock",
        },
      ],
      value: "OPT",
    };
  },
  methods: {
    // 生成指令序列
    getSeq() {
      var arr = [];
      var m = Math.floor(Math.random() * 320);
      for (var i = 0; i < 80; ++i) {
        arr.push(m);
        arr.push(Math.min(m + 1, 319));
        m = Math.floor(Math.random() * m);
        arr.push(m);
        arr.push(m + 1);
        if (m != 319) {
          m = Math.floor(Math.random() * (318 - m)) + m + 2;
        }
      }
      console.log(arr);
      return arr;
    },

    // 计算指令所在的页号
    getPage() {
      for (var i = 0; i < 320; ++i) {
        this.page[i] = Math.floor(this.addr[i] / 10);
      }
    },

    OPT() {
      this.record = [];
      var memory = [-1, -1, -1, -1];
      var count = 0;
      for (var i = 0; i < 320; ++i) {
        var inMem = false;
        for (var j = 0; j < 4; ++j) {
          if (memory[j] === this.page[i]) {
            // 页面已在内存中
            inMem = true;
            this.record.push({
              num: i,
              addr: this.addr[i],
              paddr: (this.addr[i] % 10) + j * 10,
              miss: "否",
              mem0: memory[0],
              mem1: memory[1],
              mem2: memory[2],
              mem3: memory[3],
            });
            break;
          }
        }
        if (!inMem) {
          // 页面不在内存中, 需要调页
          ++count;
          var full = true;
          for (var j = 0; j < 4; ++j) {
            if (memory[j] === -1) {
              // 内存块 j 为空, 直接调入
              full = false;
              memory[j] = this.page[i];
              this.record.push({
                num: i,
                addr: this.addr[i],
                paddr: (this.addr[i] % 10) + j * 10,
                miss: "是",
                mem0: memory[0],
                mem1: memory[1],
                mem2: memory[2],
                mem3: memory[3],
              });
              break;
            }
          }
          if (full) {
            // 内存满, 需要替换
            var replacePage = 0; // 被替换的页所在的内存块号
            var maxt = 0; // 最长的下次使用时间
            for (var j = 0; j < 4; ++j) {
              var t; // 下次使用时间
              for (t = i + 1; t < 320; ++t) {
                if (this.page[t] === memory[j]) {
                  break;
                }
              }
              if (t === 320) {
                // 之后不再使用
                replacePage = j;
                break;
              }
              if (t > maxt) {
                maxt = t;
                replacePage = j;
              }
            }
            memory[replacePage] = this.page[i]; // 替换
            this.record.push({
              num: i,
              addr: this.addr[i],
              paddr: (this.addr[i] % 10) + replacePage * 10,
              miss: "是",
              mem0: memory[0],
              mem1: memory[1],
              mem2: memory[2],
              mem3: memory[3],
            });
          }
        }
      }
      // 返回缺页次数
      return count;
    },

    FIFO() {
      this.record = [];
      var memory = [-1, -1, -1, -1];
      var queue = [];
      var count = 0;
      for (var i = 0; i < 320; ++i) {
        var inMem = false;
        for (var j = 0; j < 4; ++j) {
          if (memory[j] === this.page[i]) {
            // 页面已在内存中
            inMem = true;
            this.record.push({
              num: i,
              addr: this.addr[i],
              paddr: (this.addr[i] % 10) + j * 10,
              miss: "否",
              mem0: memory[0],
              mem1: memory[1],
              mem2: memory[2],
              mem3: memory[3],
            });
            break;
          }
        }
        if (!inMem) {
          // 页面不在内存中, 需要调页
          ++count;
          var full = true;
          for (var j = 0; j < 4; ++j) {
            if (memory[j] === -1) {
              // 内存块 j 为空, 直接调入
              full = false;
              memory[j] = this.page[i];
              queue.push(j);
              this.record.push({
                num: i,
                addr: this.addr[i],
                paddr: (this.addr[i] % 10) + j * 10,
                miss: "是",
                mem0: memory[0],
                mem1: memory[1],
                mem2: memory[2],
                mem3: memory[3],
              });
              break;
            }
          }
          if (full) {
            // 内存满, 需要替换
            var replacePage = queue.shift();
            memory[replacePage] = this.page[i]; // 替换
            queue.push(replacePage);
            this.record.push({
              num: i,
              addr: this.addr[i],
              paddr: (this.addr[i] % 10) + replacePage * 10,
              miss: "是",
              mem0: memory[0],
              mem1: memory[1],
              mem2: memory[2],
              mem3: memory[3],
            });
          }
        }
      }
      // 返回缺页次数
      return count;
    },

    LRU() {
      this.record = [];
      var memory = [-1, -1, -1, -1];
      var lastUse = [-1, -1, -1, -1];
      var count = 0;
      for (var i = 0; i < 320; ++i) {
        var inMem = false;
        for (var j = 0; j < 4; ++j) {
          if (memory[j] === this.page[i]) {
            // 页面已在内存中
            inMem = true;
            lastUse[j] = i;
            this.record.push({
              num: i,
              addr: this.addr[i],
              paddr: (this.addr[i] % 10) + j * 10,
              miss: "否",
              mem0: memory[0],
              mem1: memory[1],
              mem2: memory[2],
              mem3: memory[3],
            });
            break;
          }
        }
        if (!inMem) {
          // 页面不在内存中, 需要调页
          ++count;
          var full = true;
          for (var j = 0; j < 4; ++j) {
            if (memory[j] === -1) {
              // 内存块 j 为空, 直接调入
              full = false;
              memory[j] = this.page[i];
              lastUse[j] = i;
              this.record.push({
                num: i,
                addr: this.addr[i],
                paddr: (this.addr[i] % 10) + j * 10,
                miss: "是",
                mem0: memory[0],
                mem1: memory[1],
                mem2: memory[2],
                mem3: memory[3],
              });
              break;
            }
          }
          if (full) {
            // 内存满, 需要替换
            var replacePage = 0; // 被替换的页所在的内存块号
            var minLastUse = 320; // 最早的上次使用时间
            for (var j = 0; j < 4; ++j) {
              if (lastUse[j] < minLastUse) {
                minLastUse = lastUse[j];
                replacePage = j;
              }
            }
            memory[replacePage] = this.page[i]; // 替换
            lastUse[replacePage] = i;
            this.record.push({
              num: i,
              addr: this.addr[i],
              paddr: (this.addr[i] % 10) + replacePage * 10,
              miss: "是",
              mem0: memory[0],
              mem1: memory[1],
              mem2: memory[2],
              mem3: memory[3],
            });
          }
        }
      }
      // 返回缺页次数
      return count;
    },

    LFU() {
      this.record = [];
      var memory = [-1, -1, -1, -1];
      var useCount = [0, 0, 0, 0];
      var count = 0;
      for (var i = 0; i < 320; ++i) {
        var inMem = false;
        for (var j = 0; j < 4; ++j) {
          if (memory[j] === this.page[i]) {
            // 页面已在内存中
            inMem = true;
            ++useCount[j];
            this.record.push({
              num: i,
              addr: this.addr[i],
              paddr: (this.addr[i] % 10) + j * 10,
              miss: "否",
              mem0: memory[0],
              mem1: memory[1],
              mem2: memory[2],
              mem3: memory[3],
            });
            break;
          }
        }
        if (!inMem) {
          // 页面不在内存中, 需要调页
          ++count;
          var full = true;
          for (var j = 0; j < 4; ++j) {
            if (memory[j] === -1) {
              // 内存块 j 为空, 直接调入
              full = false;
              memory[j] = this.page[i];
              useCount[j] = 1;
              this.record.push({
                num: i,
                addr: this.addr[i],
                paddr: (this.addr[i] % 10) + j * 10,
                miss: "是",
                mem0: memory[0],
                mem1: memory[1],
                mem2: memory[2],
                mem3: memory[3],
              });
              break;
            }
          }
          if (full) {
            // 内存满, 需要替换
            var replacePage = 0; // 被替换的页所在的内存块号
            var minUse = 321; // 最少使用次数
            for (var j = 0; j < 4; ++j) {
              if (useCount[j] < minUse) {
                minUse = useCount[j];
                replacePage = j;
              }
            }
            memory[replacePage] = this.page[i]; // 替换
            useCount[j] = 1;
            this.record.push({
              num: i,
              addr: this.addr[i],
              paddr: (this.addr[i] % 10) + replacePage * 10,
              miss: "是",
              mem0: memory[0],
              mem1: memory[1],
              mem2: memory[2],
              mem3: memory[3],
            });
          }
        }
      }
      // 返回缺页次数
      return count;
    },

    Clock() {
      this.record = [];
      var memory = [-1, -1, -1, -1];
      var refbits = [0, 0, 0, 0]; // 访问位
      var count = 0; // 缺页次数
      var p = 0; // 循环队列的扫描指针
      for (var i = 0; i < 320; ++i) {
        var inMem = false;
        for (var j = 0; j < 4; ++j) {
          if (memory[j] === this.page[i]) {
            // 页面已在内存中
            inMem = true;
            refbits[j] = 1;
            this.record.push({
              num: i,
              addr: this.addr[i],
              paddr: (this.addr[i] % 10) + j * 10,
              miss: "否",
              mem0: memory[0],
              mem1: memory[1],
              mem2: memory[2],
              mem3: memory[3],
            });
            break;
          }
        }
        if (!inMem) {
          // 页面不在内存中, 需要调页
          ++count;
          var full = true;
          for (var j = 0; j < 4; ++j) {
            if (memory[j] === -1) {
              // 内存块 j 为空, 直接调入
              full = false;
              memory[j] = this.page[i];
              refbits[j] = 1;
              this.record.push({
                num: i,
                addr: this.addr[i],
                paddr: (this.addr[i] % 10) + j * 10,
                miss: "是",
                mem0: memory[0],
                mem1: memory[1],
                mem2: memory[2],
                mem3: memory[3],
              });
              break;
            }
          }
          if (full) {
            // 内存满, 需要替换
            while (1) {
              if (refbits[p] === 1) {
                refbits[p] = 0; // 访问位置为0
                p = (p + 1) % 4; // 循环指向下一个页面
              } else {
                memory[p] = this.page[i]; // 替换
                refbits[p] = 1;
                this.record.push({
                  num: i,
                  addr: this.addr[i],
                  paddr: (this.addr[i] % 10) + p * 10,
                  miss: "是",
                  mem0: memory[0],
                  mem1: memory[1],
                  mem2: memory[2],
                  mem3: memory[3],
                });
                p = (p + 1) % 4; // 循环指向下一个页面
                break;
              }
            }
          }
        }
      }
      // 返回缺页次数
      return count;
    },

    getRecord() {
      switch (this.value) {
        case "FIFO":
          this.FIFO();
          break;
        case "LRU":
          this.LRU();
          break;
        case "LFO":
          this.LFU();
          break;
        case "Clock":
          this.Clock();
          break;
        default:
          this.OPT();
      }
    },
    main() {
      this.addr = this.getSeq();
      this.getPage();

      this.result[1].count = this.FIFO();
      this.result[2].count = this.LRU();
      this.result[3].count = this.LFU();
      this.result[4].count = this.Clock();
      this.result[0].count = this.OPT();
      for (var i = 0; i < 5; ++i) {
        this.result[i].ratio = this.result[i].count / 320;
      }
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.el-container {
  margin-top: 30px;
  margin-left: 150px;
  margin-right: 150px;
}
</style>
