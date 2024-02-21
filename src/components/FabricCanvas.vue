<template>
    <div>
        <div class="container-fluid">
            <div class="row">
                <div class="splitter">
                    <div id="first">
                        <div class="p-3">
                            <h4 class="mb-3">OLevelS Card</h4>
                            <form @submit.prevent="CreateCard" method="post">
                                <div class="row gx-2">
                                    <div class="col-md-6">
                                        <div class="form-group">
                                            <label for="studentName">Student Name :</label>
                                            <input type="text" class="form-control" v-model="student.studentName"
                                                id="studentName" placeholder="Enter Name Here" required>
                                        </div>
                                    </div>
                                    <div class="col-md-6">
                                        <div class="form-group">
                                            <label for="bloodGroup">Blood Group:</label>
                                            <input type="text" class="form-control" v-model="student.bloodGroup"
                                                id="bloodGroup" placeholder="Your Blood Group Here" required>
                                        </div>
                                    </div>
                                </div>

                                <div class="row gx-2">
                                    <div class="col-md-6">
                                        <div class="form-group">
                                            <label for="fatherName">Father's Name:</label>
                                            <input type="text" class="form-control" v-model="student.fatherName"
                                                id="fatherName" placeholder="Your Father Name Here" required>
                                        </div>
                                    </div>
                                    <div class="col-md-6">
                                        <div class="form-group">
                                            <label for="motherName">Mother's Name:</label>
                                            <input type="text" class="form-control" v-model="student.motherName"
                                                id="motherName" placeholder="Your Mother Name Here" required>
                                        </div>
                                    </div>
                                </div>

                                <div class="form-group">
                                    <label for="address">Address:</label>
                                    <input type="text" class="form-control" v-model="student.address" id="address"
                                        placeholder="Your Address Here" required>
                                </div>

                                <div class="row g-0">
                                    <div class="col-md-1 mr-20">
                                        <div class="form-group">
                                            <label for="studentImage">Select image:</label>
                                            <canvas ref="img"></canvas>
                                            <input type="file" id="studentImage" name="studentImage"
                                                @change="handleImageChange" accept="image/*">
                                        </div>
                                    </div>
                                </div>

                                <div class="form-group">
                                    <br />
                                    <button @click.prevent="CreateCard">Create Card To PDF</button>
                                    <br />
                                    <br />
                                    <button @click.prevent="CreateCardSVG">Create Card To SVG</button>

                                    <!-- <button type="submit" class="step-form-submit1">DOWNLOAD</button> -->
                                </div>
                            </form>
                        </div>
                    </div>
                    <div id="separator"></div>
                    <div id="preview-screen">
                        <canvas id="studentcard" width="142" height="213"></canvas>
                    </div>
                </div>

            </div>

        </div>
    </div>
</template>
<script>
import '/public/css/style.css';
import '/public/css/bootstrap.min.css';
import '/public/js/vendor/jquery-3.6.0.js';
import '/public/js/vendor/bootstrap.min.js';
import '/public/js/vendor/bootstrap.bundle.min.js';
// import * as Masonry from '/public/js/vendor/masonry.pkgd.js';
import svgToPdf from 'svg-to-pdfkit';
import jsPDF from 'jspdf';

import 'jspdf-autotable';
import html2pdf from 'html2pdf.js';

import { fabric } from "fabric";
import html2canvas from 'html2canvas'; // Import html2canvas for rendering SVG to canvas

import axios from "axios";
axios.defaults.headers.common['Access-Control-Allow-Origin'] = '*';
let studentcard, rootEl;

export default {
    data() {
        return {
            student: {
                studentName: "",
                bloodGroup: "",
                fatherName: "",
                motherName: "",
                address: "",
                studentImage: null,
            },
            image: null
        }
    },
    mounted() {
        this.init();
        this.studentImage = new fabric.Canvas(this.$refs.studentImage);
    },
    methods: {
        init() {
            rootEl = this;
            studentcard = new fabric.StaticCanvas('studentcard');
            this.loadBackground(studentcard, '../branding/theme/studentcard.svg');
        },
        changeText(card, from, to) {
            card.forEach(element => {
                if (element.text == from) {
                    element.text = to;
                }
            });
        },
        loadBackground(canvas, image, arr = "") {
            return new Promise((resolve, reject) => {
                fabric.loadSVGFromURL(image, function (objects, options) {
                    var dollars = fabric.util.groupSVGElements(objects, options);
                    let card = dollars._objects;
                    if (arr != "") {
                        if (arr.studentName != "Your Name Here") {
                            rootEl.changeText(card, "Your Name Here", arr.studentName);
                        }
                        if (arr.bloodGroup != "Your Blood Group Here") {
                            rootEl.changeText(card, "Your Blood Group Here", arr.bloodGroup);
                        }
                        if (arr.fatherName != "Your Father Name Here") {
                            rootEl.changeText(card, "Your Father Name Here", arr.fatherName);
                        }
                        if (arr.motherName != "Your Mother Name Here") {
                            rootEl.changeText(card, "Your Mother Name Here", arr.motherName);
                        }
                        if (arr.address != "Your Address Here") {
                            rootEl.changeText(card, "Your Address Here", arr.address);
                        }
                    }
                    canvas.centerObject(dollars);
                    canvas.setBackgroundImage(dollars);
                    canvas.calcOffset();
                    canvas.renderAll();
                    resolve();
                });
            });
        },

        CreateCardSVG(e) {
            this.loadBackground(studentcard, './branding/theme/studentcard.svg', this.student)
                .then(() => {
                    this.downloadCardSVG(studentcard);
                });
        },

        CreateCard(e) {
            this.loadBackground(studentcard, './branding/theme/studentcard.svg', this.student)
                .then(() => {
                    this.downloadCard(studentcard);
                });
        },
        downloadCardSVG(canvas, name = 'studentcard.svg') {
            var filedata = canvas.toSVG();
            var blob = new Blob([filedata], { type: "image/svg+xml;charset=utf-8" });
            const blobUrl = URL.createObjectURL(blob);
            const link = document.createElement("a");
            link.href = blobUrl;
            link.download = name;
            document.body.appendChild(link);
            link.dispatchEvent(
                new MouseEvent('click', {
                    bubbles: true,
                    cancelable: true,
                    view: window
                })
            );
            document.body.removeChild(link);
        },



        // downloadCard(canvas, name = 'studentcard') {
        //     console.log('canvas', canvas);
        //     const filename = name + '.pdf';
        //     const options = {
        //         filename: filename,
        //         image: { type: 'jpeg', quality: 1 },
        //         html2canvas: { scale: 1, logging: true, width: 142, height: 213, dpi: 300 },
        //         jsPDF: { unit: 'px', format: [142, 213], orientation: 'portrait' }
        //     };
        //     // document.body.style.width = '142px';
        //     // document.body.style.height = '213px';


        //     // Get the canvas element
        //     const canvasElement = document.getElementById('studentcard');

        //     // Generate PDF from canvas element
        //     html2pdf().from(canvasElement).set(options).save();


        //     // const content = this.$el.querySelector('#studentcard');
        //     // console.log('this.$el', this.$el.querySelector('#studentcard'))
        //     // console.log('content', content)

        //     // // Generate PDF from HTML content
        //     // console.log('canvas.lowerCanvasEl', html2pdf().from(content).set(options));
        //     // html2pdf().from(content).set(options).save();
        //     // Generate PDF
        //     // html2pdf().from(canvas.lowerCanvasEl).set(options).save();
        // },




        downloadCard(canvas, name = 'studentcard.pdf') {
            var tempCanvas = document.createElement("canvas");
            var ctx = tempCanvas.getContext("2d");
            var img = new Image();
            const scale = 2;
            tempCanvas.width = canvas.width * scale;
            tempCanvas.height = canvas.height * scale;
            img.onload = function () {
                ctx.drawImage(img, 0, 0, tempCanvas.width, tempCanvas.height);
                var imageData = tempCanvas.toDataURL("image/jpeg", 1.0);
                var pdf = new jsPDF('p', 'mm', [canvas.width, canvas.height]);
                pdf.addImage(imageData, 'JPEG', 0, 0, canvas.width, canvas.height);
                pdf.save(name);
            };
            var filedata = canvas.toSVG();
            img.src = 'data:image/svg+xml;base64,' + btoa(unescape(encodeURIComponent(filedata)));
        }


        ,


        downloadCards() {
            setTimeout(function () {
                rootEl.downloadCard(studentcard);
            }, 3000);
        },
        handleImageChange(event) {
            const file = event.target.files[0];
            const reader = new FileReader();
            reader.onload = (event) => {
                const imageUrl = event.target.result;
                this.student.studentImage = imageUrl;
                this.replaceClippingImage(imageUrl);

            };
            reader.readAsDataURL(file);
        },
        replaceClippingImage(imageUrl) {
            const containerWidth = 50;
            const containerHeight = 50;
            const radius = 25;

            // const imgCanvas = this.$refs.img;
            const imgCanvas = document.createElement("canvas");
            imgCanvas.width = containerWidth;
            imgCanvas.height = containerHeight;
            const imgCtx = imgCanvas.getContext('2d');
            imgCtx.clearRect(0, 0, imgCanvas.width, imgCanvas.height);

            const img = new Image();
            img.onload = () => {
                imgCtx.clearRect(0, 0, imgCanvas.width, imgCanvas.height);
                imgCtx.beginPath();
                imgCtx.arc(radius, radius, radius, 0, Math.PI * 2, true); // draw circle
                imgCtx.closePath();
                imgCtx.clip();

                // Calculate scaling factor to fit the image within the circle
                const scalingFactor = Math.min(containerWidth / img.width, containerHeight / img.height);

                // Draw the image inside the circle
                const scaledWidth = img.width * scalingFactor;
                console.log('scaledWidth', scaledWidth);
                const scaledHeight = img.height * scalingFactor;
                console.log('scaledHeight', scaledHeight);
                const offsetX = (containerWidth - scaledWidth) / 2;
                console.log('offsetX', offsetX);
                const offsetY = (containerHeight - scaledHeight) / 2;
                console.log('offsetY', offsetY);
                imgCtx.drawImage(img, offsetX, offsetY, scaledWidth, scaledHeight);

                // Convert canvas to base64 URL
                const clippedImageUrl = imgCanvas.toDataURL();
                console.log('clippedImageUrl', clippedImageUrl);
                fabric.Image.fromURL(clippedImageUrl, (fabricImg) => {
                    const circle = new fabric.Circle({
                        radius: radius,
                        fill: 'transparent',
                        stroke: '#E34750',
                        strokeWidth: 4,
                        left: 69,
                        top: 73,
                        originX: 'center',
                        originY: 'center',
                    });
                    console.log('circle.width', clippedImageUrl);
                    console.log('img.width', img.width);
                    const scaleFactor = radius * 2 / Math.max(fabricImg.width, fabricImg.height);
                    fabricImg.scale(scaleFactor);
                    console.log('scalingFactor', scalingFactor);
                    fabricImg.set({
                        left: circle.left,
                        top: circle.top,
                        originX: 'center',
                        originY: 'center',
                    });
                    studentcard.add(fabricImg, circle);
                    studentcard.renderAll();
                });
            }, { crossOrigin: 'anonymous' };
            img.src = imageUrl;
        }

        // end adding student image
    },
    watch: {
        student: {
            deep: true,
            handler(newStudentData) {
                console.log(newStudentData);
                this.loadBackground(studentcard, './branding/theme/studentcard.svg', newStudentData);
            }
        }
    },
};
</script>

  
<style scoped>
.canvas {
    border: 1px solid #ccc;
}

.splitter {
    width: 100%;
    height: 100%;
    display: flex;
    padding: 0;
}

#separator {
    cursor: col-resize;
    background: #7f7f7f;
    background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='10' height='30'><path d='M2 0 v30 M5 0 v30 M8 0 v30' fill='none' stroke='black'/></svg>");
    background-repeat: no-repeat;
    background-position: center;
    width: 10px;
    /* Prevent the browser's built-in drag from interfering */
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}

#first {
    width: 50%;
    min-width: 528px;
    max-width: 50%;
    background: #dadce0;
}

#preview-screen {
    width: 50%;
    min-width: 707px;
    max-width: 100%;
}

.step-input2 {
    margin-bottom: 10px;

}

/*submit button*/

.step-form-submit1 {
    color: #fff;
    position: relative;
    background: #d92d2d;
    border: 0;
    outline: 0;
    padding: 6px 12px;
    z-index: 2;
    font-weight: 700;
    text-transform: uppercase;
    border-radius: 6px;
    margin-top: 10px;
}

.grid {
    width: 100%;
}

.grid-item {
    float: left;
    width: 22em;
    height: 33em;
}

.grid-item img {
    max-width: 100%;
    height: 100%;
    object-fit: cover;
}

.grid-item--width2 {
    width: 44em;
}

.grid-item--height2 {
    height: 50%;
}

.primary-color {
    border: 0;
    padding: 0;
    width: 100%;
    height: 100%;
    cursor: pointer;
    transform: translate(2%, 0%);
}

input[type='color'] {
    -webkit-appearance: none;
    padding: 0;
    border: none;
    border-radius: 100%;
    width: 40px;
    height: 40px;
    border: 3px solid #000;
}

input[type='color']::-webkit-color-swatch {
    border: 0px solid #000;
    border-radius: 100%;
    padding: 0;
}

input[type='color']::-webkit-color-swatch-wrapper {
    border: 0px solid #000;
    border-radius: 100%;
    padding: 0;
}

.mr-20 {
    margin-right: 10px;
}

.mr-20:nth-child(1) {
    margin-right: 30px;
}

.mr-20:nth-child(2) {
    margin-right: 40px;
}
</style>