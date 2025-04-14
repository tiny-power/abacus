<template>
    <div>
        <div class="titlebar">Abacus</div>
        <div style="display: flex" class="drag">
            <div style="width: 100%; height: 422px; overflow: auto" class="no-drag">
                <textarea id="codeMirror"></textarea>
            </div>
            <div
                ref="scrollContainer"
                class="no-drag"
                style="
                    overflow: hidden;
                    height: 422px;
                    background: #1f2430;
                    width: 200px;
                    color: #3ad900;
                    text-align: right;
                    padding: 3px 10px 0px 0px;
                    box-sizing: border-box;
                "
            >
                <div
                    v-for="(item, index) in output"
                    :key="index"
                    class="ellipsis"
                    v-html="item"
                    @click="copyToClipboard(item)"
                ></div>
            </div>
        </div>
    </div>
</template>

<script>
import * as math from 'mathjs'
import CodeMirror from 'codemirror'
import 'codemirror/mode/javascript/javascript.js'
import 'codemirror/addon/display/placeholder.js'
import 'codemirror/lib/codemirror.css'
import 'codemirror/theme/ayu-mirage.css'
export default {
    data() {
        return {
            value: '',
            editor: null,
            output: []
        }
    },
    mounted() {
        this.codeMirrorInit()
    },
    methods: {
        codeMirrorInit() {
            const codeMirror = document.getElementById('codeMirror')
            this.editor = CodeMirror.fromTextArea(codeMirror, {
                theme: 'ayu-mirage',
                lineNumbers: true,
                line: true,
                tabSize: 4,
                indentUnit: 4,
                mode: 'javascript',
                lineWrapping: true,
                autofocus: true
            })

            if (this.value) {
                this.editor.setValue(this.value)
            }

            this.editor.on('change', () => {
                this.value = this.editor.getValue()
                this.doMath(this.editor.getValue())
            })
            this.editor.on('scroll', instance => {
                this.$refs.scrollContainer.scrollTop = instance.doc.scrollTop
            })
        },
        doMath(input) {
            let output = []
            let scope = {}
            for (const line of input.split('\n')) {
                if (line) {
                    if (line.startsWith('//')) {
                        output.push('&nbsp;')
                    } else {
                        try {
                            const value = math.evaluate(line, scope)
                            output.push(math.format(value, { upperExp: 11, precision: 14 }))
                        } catch (e) {
                            output.push('&nbsp;')
                        }
                    }
                } else {
                    output.push('&nbsp;')
                }
            }
            this.output = output
        },
        async copyToClipboard(text) {
            try {
                await navigator.clipboard.writeText(text)
                this.$message({
                    message: 'Copied',
                    type: 'success'
                })
            } catch (err) {
                console.error(err)
            }
        }
    }
}
</script>

<style scoped>
.titlebar {
    height: 28px;
    user-select: none;
    -webkit-app-region: drag;
    background-color: #1f2430;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #fff;
    font-size: 14px;
}
::v-deep .CodeMirror {
    height: 100% !important;
}
::v-deep .CodeMirror-linenumber {
    text-align: center !important;
}
::v-deep .CodeMirror pre.CodeMirror-line,
::v-deep.CodeMirror pre.CodeMirror-line-like {
    font-size: 16px;
}
.ellipsis {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    height: 18.5px;
    line-height: 18.5px;
    font-size: 16px;
    font-family: monospace;
    cursor: pointer;
}
</style>
