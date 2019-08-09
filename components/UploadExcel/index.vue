<template>
  <div style="display: inline-block;">
    <input ref="excel-upload-input" class="excel-upload-input" type="file" accept=".xlsx, .xls" @change="handleClick">
    <el-button :loading="loading" type="info" @click="handleUpload">
      {{ btnName }}
    </el-button>
  </div>
</template>

<script>
import XLSX from 'xlsx'

export default {
  props: {
    beforeUpload: Function, // eslint-disable-line
    onSuccess: Function, // eslint-disable-line
    btnName: String
  },
  data () {
    return {
      loading: false,
      excelData: {
        header: null,
        results: null
      }
    }
  },
  methods: {
    generateData ({ header, results }) {
      this.excelData.header = header
      this.excelData.results = results
      this.onSuccess && this.onSuccess(this.excelData)
    },
    handleDrop (e) {
      e.stopPropagation()
      e.preventDefault()
      if (this.loading) return
      const files = e.dataTransfer.files
      if (files.length !== 1) {
        this.$message.error('Only support uploading one file!')
        return
      }
      const rawFile = files[0] // only use files[0]

      if (!this.isExcel(rawFile)) {
        this.$message.error('Only supports upload .xlsx, .xls, .csv suffix files')
        return false
      }
      this.upload(rawFile)
      e.stopPropagation()
      e.preventDefault()
    },
    handleDragover (e) {
      e.stopPropagation()
      e.preventDefault()
      e.dataTransfer.dropEffect = 'copy'
    },
    handleUpload () {
      this.$refs['excel-upload-input'].click()
    },
    handleClick (e) {
      const files = e.target.files
      const rawFile = files[0] // only use files[0]
      if (!rawFile) return
      this.upload(rawFile)
    },
    upload (rawFile) {
      this.$refs['excel-upload-input'].value = null // fix can't select the same excel

      if (!this.beforeUpload) {
        this.readerData(rawFile)
        return
      }
      const before = this.beforeUpload(rawFile)
      if (before) {
        this.readerData(rawFile)
      }
    },
    readerData (rawFile) {
      this.loading = true
      return new Promise((resolve, reject) => {
        const reader = new FileReader()
        reader.onload = e => {
          const data = e.target.result
          const workbook = XLSX.read(data, { type: 'array' })
          const firstSheetName = workbook.SheetNames[0]
          const worksheet = workbook.Sheets[firstSheetName]
          const header = this.getHeaderRow(worksheet)
          // const results = XLSX.utils.sheet_to_json(worksheet)
          const results = this.getResultsJson(worksheet)
          this.generateData({ header, results })
          this.loading = false
          resolve()
        }
        reader.readAsArrayBuffer(rawFile)
      })
    },
    getHeaderRow (sheet) {
      const headers = []
      const range = XLSX.utils.decode_range(sheet['!ref'])
      console.log(range)
      let C
      const R = range.s.r
      /* start in the first row */
      for (C = range.s.c; C <= range.e.c; ++C) { /* walk every column in the range */
        const cell = sheet[XLSX.utils.encode_cell({ c: C, r: R })]
        /* find the cell in the first row */
        let hdr = 'UNKNOWN ' + C // <-- replace with your desired default
        if (cell && cell.t) hdr = XLSX.utils.format_cell(cell)
        headers.push(hdr)
      }
      return headers
    },
    getResultsJson (sheet, opts) {
      if (sheet == null || sheet['!ref'] == null) return []
      var val = {t: 'n', v: 0}
      var header = 0
      var offset = 1
      var hdr = []
      var v = 0
      var vv = ''
      var r = { s: { r: 0, c: 0 }, e: { r: 0, c: 0 } }
      var o = opts || {}
      var range = o.range != null ? o.range : sheet['!ref']
      if (o.header === 1) header = 1
      else if (o.header === 'A') header = 2
      else if (Array.isArray(o.header)) header = 3
      switch (typeof range) {
      case 'string': r = this.safe_decode_range(range); break
      case 'number': r = this.safe_decode_range(sheet['!ref']); r.s.r = range; break
      default: r = range
      }
      if (header > 0) offset = 0
      var rr = XLSX.utils.encode_row(r.s.r)
      var cols = []
      var out = []
      var outi = 0
      var counter = 0
      var dense = Array.isArray(sheet)
      var R = r.s.r
      var C = 0
      var CC = 0
      if (dense && !sheet[R]) sheet[R] = []
      for (C = r.s.c; C <= r.e.c; ++C) {
        cols[C] = XLSX.utils.encode_col(C)
        val = dense ? sheet[R][C] : sheet[cols[C] + rr]
        switch (header) {
        case 1: hdr[C] = C - r.s.c; break
        case 2: hdr[C] = cols[C]; break
        case 3: hdr[C] = o.header[C - r.s.c]; break
        default:
          if (val == null) val = { w: '__EMPTY', t: 's' }
          vv = v = XLSX.utils.format_cell(val, null, o)
          counter = 0
          for (CC = 0; CC < hdr.length; ++CC) if (hdr[CC] === vv) vv = v + '_' + (++counter)
          hdr[C] = vv
        }
      }
      for (R = r.s.r + offset; R <= r.e.r; ++R) {
        var row = this.make_json_row(sheet, r, R, cols, header, hdr, dense, o)
        if ((row.isempty === false) || (header === 1 ? o.blankrows !== false : !!o.blankrows)) out[outi++] = row.row
      }
      out.length = outi
      return out
    },
    safe_decode_range (range) {
      var o = { s: { c: 0, r: 0 }, e: { c: 0, r: 0 } }
      var idx = 0
      var i = 0
      var cc = 0
      var len = range.length
      for (idx = 0; i < len; ++i) {
        if ((cc = range.charCodeAt(i) - 64) < 1 || cc > 26) break
        idx = 26 * idx + cc
      }
      o.s.c = --idx

      for (idx = 0; i < len; ++i) {
        if ((cc = range.charCodeAt(i) - 48) < 0 || cc > 9) break
        idx = 10 * idx + cc
      }
      o.s.r = --idx

      if (i === len || range.charCodeAt(++i) === 58) { o.e.c = o.s.c; o.e.r = o.s.r; return o }

      for (idx = 0; i !== len; ++i) {
        if ((cc = range.charCodeAt(i) - 64) < 1 || cc > 26) break
        idx = 26 * idx + cc
      }
      o.e.c = --idx

      for (idx = 0; i !== len; ++i) {
        if ((cc = range.charCodeAt(i) - 48) < 0 || cc > 9) break
        idx = 10 * idx + cc
      }
      o.e.r = --idx
      return o
    },
    make_json_row (sheet, r, R, cols, header, hdr, dense, o) {
      var rr = XLSX.utils.encode_row(R)
      var defval = o.defval
      var raw = o.raw || !o.hasOwnProperty('raw')
      var isempty = true
      var row = (header === 1) ? [] : {}
      if (header !== 1) {
        if (Object.defineProperty) {
          try {
            Object.defineProperty(row, '__rowNum__', { value: R, enumerable: false })
          } catch (e) { row.__rowNum__ = R }
        } else { row.__rowNum__ = R }
      }
      if (!dense || sheet[R]) {
        for (var C = r.s.c; C <= r.e.c; ++C) {
          var val = dense ? sheet[R][C] : sheet[cols[C] + rr]
          if (val === undefined || val.t === undefined) {
            if (defval === undefined) continue
            if (hdr[C] !== null) { row[hdr[C]] = defval }
            continue
          }
          var v = typeof (val.v) === 'number' ? val.w : val.v
          switch (val.t) {
          case 'z': if (v == null) break; continue
          case 'e': v = void 0; break
          case 's': case 'd': case 'b': case 'n': break
          default: throw new Error('unrecognized type ' + val.t)
          }
          if (hdr[C] !== null) {
            if (v == null) {
              if (defval !== undefined) row[hdr[C]] = defval
              else if (raw && v === null) row[hdr[C]] = null
              else continue
            } else {
              row[hdr[C]] = raw ? v : XLSX.utils.format_cell(val,v,o)
            }
            if (v !== null) isempty = false
          }
        }
        return { row: row, isempty: isempty }
      }
    },
    isExcel (file) {
      return /\.(xlsx|xls|csv)$/.test(file.name)
    }
  }
}
</script>

<style scoped>
.excel-upload-input{
  display: none;
  z-index: -9999;
}
.drop{
  border: 2px dashed #bbb;
  width: 600px;
  height: 160px;
  line-height: 160px;
  margin: 0 auto;
  font-size: 24px;
  border-radius: 5px;
  text-align: center;
  color: #bbb;
  position: relative;
}
</style>
