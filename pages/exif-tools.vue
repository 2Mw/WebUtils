<template>
    <ClientOnly>
        <p class="text-xl text-opacity-10 opacity-50">
            <UTooltip class="max-w-max" :ui="{ width: 'max-w-none', lineClamp: 2 }">
                <template #text>
                    <span>{{ exifText }}</span>
                </template>
                EXIF
            </UTooltip>
            Tools
        </p>
        <Hr />
        <div class="flex relative items-center justify-center z-0 h-40">
            <input type="file" @change="submitImg" @drag="submitImg"
                class="absolute left-0 top-0 inline-block file:hidden text-red-50 text-opacity-0 z-10 bg-opacity-0 w-full h-full cursor-grab peer/fip border border-dashed border-slate-400 border-opacity-30 hover:border-opacity-90" />
            <div class="text-center opacity-40 peer-hover/fip:opacity-100">
                Click to select image <br /> OR <br /> Drag image to here
            </div>
            <img :src="imgURL" class="absolute self-center h-full brightness-50" />
        </div>
        <p class="text-sm text-center text-slate-400 my-2" v-if="imgName.length > 0">{{ imgName }}</p>
        <p class="text-sm text-center text-slate-400 my-2" v-if="imgName.length > 0">
            <UButton variant="outline" size="xs" class="text-xs" @click="clearExif">Clear EXIF</UButton>
            <a :href="gpsLink" v-if="gpsLink.length > 0" target="_blank" class="ml-3">
                <UButton variant="outline" color="red" size="xs">This Position!!</UButton>
            </a>
        </p>
        <Hr />
        <div class="grid lg:grid-cols-3 gap-6 grid-cols-1">
            <EntryTable v-if="basicInfoTable.length > 0" caption="Basic Info" :rows="basicInfoTable" />
            <EntryTable v-if="deviceInfoTable.length > 0" caption="Device Info" :rows="deviceInfoTable" />
            <EntryTable v-if="exifInfoTable.length > 0" caption="Exif Info" :rows="exifInfoTable" />
            <EntryTable v-if="gpsInfoTable.length > 0" caption="GPS Info" :rows="gpsInfoTable" />
            <EntryTable v-if="ifd1InfoTable.length > 0" caption="ifd1 Info" :rows="ifd1InfoTable" />
            <EntryTable v-if="iccInfoTable.length > 0" caption="ICC Info" :rows="iccInfoTable" />
            <EntryTable v-if="jijfInfoTable.length > 0" caption="JIJF Info" :rows="jijfInfoTable" />
            <EntryTable v-if="ihdrInfoTable.length > 0" caption="IHDR Info" :rows="ihdrInfoTable" />
            <div v-if="mergeInfo.hasOwnProperty('makerNote')">
                <p class="text-slate-300 pb-2 text-sm caption-top">MarkerNotes</p>
                <p class="break-words text-slate-400 text-xs">
                    {{ mergeInfo.makerNote }}
                </p>
            </div>
            <EntryTable v-if="rawInfoTable.length > 0" caption="Raw Info" :rows="rawInfoTable" />
        </div>
    </ClientOnly>
</template>

<script setup lang="ts">
import exifr, { Exifr } from 'exifr';
import { Entry } from '~/composables/models/Entry';
const exifText = "Exchangeable image file format，包含数码照片的属性信息和拍摄数据。"

useHead({
    title: 'Exif Tool'
})

//  GoogleMap: https://www.google.com/maps/@($latitude),($longtitude44),15z

const imgURL = ref('');
const imgName = ref('');
const imgFile = ref({} as File);

const gpsLink = ref('');

const defaultOptions = {
    // Segments (JPEG APP Segment, PNG Chunks, HEIC Boxes, etc...)
    tiff: true,
    ifd0: true, // aka image
    ifd1: true, // aka thumbnail
    exif: true,
    gps: true,
    interop: true,
    // Other TIFF tags
    makerNote: true,
    userComment: true,

    xmp: true,
    icc: true,
    iptc: true,
    jfif: true, // (jpeg only)
    ihdr: true, // (png only)
    // Formatters
    sanitize: true,
    mergeOutput: false,
    // Chunked reader
    chunkSize: 65536, // 64kb
    chunkLimit: 5,
}

let rawInfo = ref({} as any)
let mergeInfo = ref({} as any)
// 基本信息
let basicInfoTable = ref([] as Entry[])
// 设备信息
let deviceInfoTable = ref([] as Entry[]);
// GPS信息
let gpsInfoTable = ref([] as Entry[]);
// Exif信息
let exifInfoTable = ref([] as Entry[]);
// ICC 信息
let iccInfoTable = ref([] as Entry[]);
// jijf 信息
let jijfInfoTable = ref([] as Entry[]);
// iHDR 信息
let ihdrInfoTable = ref([] as Entry[]);
// ifd1
let ifd1InfoTable = ref([] as Entry[]);
// raw
let rawInfoTable = ref([] as Entry[]);


const basicInfoKeys = [
    'CreateDate', 'Model', 'Software', 'latitude', 'longitude'
]


watch(mergeInfo, () => {
    basicInfoTable.value.splice(0, basicInfoTable.value.length);
    // '', '', , '', 
    let sizeEntry = new Entry('ImageSize', '');
    let flag = true;
    if (mergeInfo.value.hasOwnProperty('ExifImageHeight')) sizeEntry.value = `${mergeInfo.value['ExifImageWidth']} x ${mergeInfo.value['ExifImageHeight']}`;
    else if (mergeInfo.value.hasOwnProperty('ImageWidth')) sizeEntry.value = `${mergeInfo.value['ImageWidth']} x ${mergeInfo.value['ImageHeight']} `;
    else flag = false;
    if (flag) basicInfoTable.value.push(sizeEntry);

    for (let key of basicInfoKeys) {
        if (mergeInfo.value.hasOwnProperty(key)) {
            basicInfoTable.value.push(new Entry(key, mergeInfo.value[key]));
        }
    }

    // raw
    rawInfoTable.value.splice(0, rawInfoTable.value.length);
    for (let key in mergeInfo.value) {
        rawInfoTable.value.push(new Entry(key, mergeInfo.value[key]));
    }
});

watch(rawInfo, () => {
    // device
    deviceInfoTable.value.splice(0, deviceInfoTable.value.length);
    if (rawInfo.value.hasOwnProperty('ifd0')) {
        let deviceObj = rawInfo.value['ifd0'];
        for (let key in deviceObj) {
            deviceInfoTable.value.push(new Entry(key, deviceObj[key]));
        }
    }

    // exif
    exifInfoTable.value.splice(0, exifInfoTable.value.length);
    if (rawInfo.value.hasOwnProperty('exif')) {
        let exifObj = rawInfo.value['exif'];

        for (let key in exifObj) {
            exifInfoTable.value.push(new Entry(key, exifObj[key]));
        }
    }

    // exif
    gpsInfoTable.value.splice(0, gpsInfoTable.value.length);
    gpsLink.value = '';
    if (rawInfo.value.hasOwnProperty('gps')) {
        let gps = rawInfo.value['gps'];

        for (let key in gps) {
            gpsInfoTable.value.push(new Entry(key, gps[key]));
        }

        if (gps.hasOwnProperty('latitude')) {
            gpsLink.value = `https://www.google.com/maps/@${gps['latitude']},${gps['longitude']},15z`
        }
    }

    // icc
    iccInfoTable.value.splice(0, iccInfoTable.value.length);
    if (rawInfo.value.hasOwnProperty('icc')) {
        let icc = rawInfo.value['icc'];

        for (let key in icc) {
            iccInfoTable.value.push(new Entry(key, icc[key]));
        }
    }

    ifd1InfoTable.value.splice(0, ifd1InfoTable.value.length);
    if (rawInfo.value.hasOwnProperty('ifd1')) {
        let ifd1 = rawInfo.value['ifd1'];
        for (let key in ifd1) {
            ifd1InfoTable.value.push(new Entry(key, ifd1[key]));
        }
    }

    // jijf
    jijfInfoTable.value.splice(0, jijfInfoTable.value.length);
    if (rawInfo.value.hasOwnProperty('jijf')) {
        let jijf = rawInfo.value['jijf'];

        for (let key in jijf) {
            jijfInfoTable.value.push(new Entry(key, jijf[key]));
        }
    }

    // ihdr
    ihdrInfoTable.value.splice(0, ihdrInfoTable.value.length);
    if (rawInfo.value.hasOwnProperty('ihdr')) {
        let ihdr = rawInfo.value['ihdr'];

        for (let key in ihdr) {
            ihdrInfoTable.value.push(new Entry(key, ihdr[key]));
        }
    }
});

const submitImg = async (e: any) => {
    imgFile.value = e.target.files[0];
    if (imgFile.value) {
        imgName.value = imgFile.value.name;
        const reader = new FileReader();
        reader.addEventListener('load', () => {
            if (typeof reader.result === 'string') imgURL.value = reader.result;
        })
        reader.readAsDataURL(imgFile.value);
        rawInfo.value = await exifr.parse(imgFile.value, defaultOptions);
        mergeInfo.value = await exifr.parse(imgFile.value, { ...defaultOptions, mergeOutput: true });

        // console.log('rawinfo:', rawInfo.value);
        // console.log('mergeInfo:', mergeInfo.value);
    }
}


const clearExif = async () => {
    const reader = new FileReader();

    // refer: https://jsfiddle.net/mowglisanu/frhwm2xe/3/
    reader.addEventListener('load', () => {
        if (reader.result != null && reader.result instanceof ArrayBuffer) {
            var dv = new DataView(reader.result);
            var offset = 0, recess = 0;
            var pieces = [];
            var i = 0;
            if (dv.getUint16(offset) == 0xffd8) {
                offset += 2;
                var app1 = dv.getUint16(offset);
                offset += 2;
                while (offset < dv.byteLength) {
                    if (app1 == 0xffe1 || app1 == 0xffe2) {
                        pieces[i] = { recess: recess, offset: offset - 2 };
                        recess = offset + dv.getUint16(offset);
                        i++;
                    }
                    else if (app1 == 0xffda) {
                        break;
                    }
                    offset += dv.getUint16(offset);
                    var app1 = dv.getUint16(offset);
                    offset += 2;
                }
                if (pieces.length > 0) {
                    var newPieces = [];
                    pieces.forEach(function (v) {
                        newPieces.push(reader.result?.slice(v.recess, v.offset));
                    }, this);
                    newPieces.push(reader.result.slice(recess));
                    var br = new Blob(newPieces, { type: 'image/jpeg' });
                    let clearedURL = URL.createObjectURL(br)
                    console.log(clearedURL);

                    var a = document.createElement('a');
                    a.href = clearedURL;
                    a.download = crypto.randomUUID();
                    a.click();
                }
            }
        }
    });

    reader.readAsArrayBuffer(imgFile.value);
}

</script>