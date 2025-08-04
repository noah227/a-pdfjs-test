<template>
    <div id="render-area">
        <div ref="refContainer">

        </div>
        <div ref="refContainerTextContent"></div>
    </div>
</template>

<script lang="ts" setup>
import {AnnotationMode, PDFDocumentLoadingTask, PDFPageProxy, TextLayer} from "pdfjs-dist";
import {onMounted, ref} from "vue";


const refContainer = ref()
const refContainerTextContent = ref()
const pdfjs = require("pdfjs-dist")
console.log(pdfjs)

// 要先指定worker地址（这里把相关文件拷贝到根目录下了）
pdfjs.GlobalWorkerOptions.workerSrc = "/build/pdf.worker.mjs"

const loadPdf = () => {
    refContainer.value.innerHTML = ""
    const loadingTask: PDFDocumentLoadingTask = pdfjs.getDocument("/JavaDoc.pdf")
    loadingTask.promise.then((pdf) => {
        if (pdf) {
            console.log(pdf.numPages)
            // 这里没有做任何性能处理，本测试用文档50多页，加载也很流畅的
            for (let i = 0; i < pdf.numPages; i++) {
            // 测试配置
            // for (let i = 5; i < 6; i++) {
                console.log(`第${i}页`)
                pdf.getPage(i).then(page => {
                    renderPage(page)
                    // renderPageTextContent(page)
                }).catch(() => {
                    console.log("加载结束")
                })
            }
        }
    })
}

/**
 * 这里只是简单创建canvas并渲染
 * 当然也可以封装到组件，绑定page实例，组件内处理渲染，这样其实更好
 * @param page
 */
const renderPage = (page: PDFPageProxy) => {
    const canvas = document.createElement("canvas")
    const context = canvas.getContext("2d")
    const viewport = page.getViewport({scale: 1.5})
    canvas.width = viewport.width
    canvas.height = viewport.height

    page.render({
        canvas,
        canvasContext: context as CanvasRenderingContext2D,
        annotationMode: AnnotationMode.ENABLE,
        isEditing: true,
        viewport
    })
    refContainer.value.appendChild(canvas)
}

const renderPageTextContent = (page: PDFPageProxy) => {
    // 为文本层也创建一个容器
    const div = document.createElement("div")
    div.className = "render-container--text-content"
    const viewport = page.getViewport({scale: 1.5})
    // 容器本身尺寸不用定义，因为render函数会自动处理
    const textLayer = new TextLayer({
        textContentSource: page.streamTextContent(),
        viewport,
        container: div,
    });
    textLayer.render();
    refContainerTextContent.value.appendChild(div)
}

onMounted(() => {
    loadPdf()
})
</script>

<style lang="scss" scoped>
#render-area {
    width: 100%;
    height: 100%;
    position: relative;

    > div {
        position: absolute;
        left: 0;
        right: 0;
        top: 0;
        height: fit-content;
        > * {
            margin: 0 auto;
        }
    }
}
</style>
