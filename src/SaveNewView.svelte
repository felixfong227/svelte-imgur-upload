<script>
    import { imagesStore } from './store.js';

    export let skip_saving = false;
    export let DB;
    export let clientID;

    let uploading = false;

    let images = [];

    imagesStore.subscribe(imagesFromStore => {
        return (images = imagesFromStore || []);
    });

    async function encodeImage(file) {
        return new Promise((resolve, reject) => {
            let reader = new FileReader();
            reader.onloadend = () => resolve(reader.result);
            reader.onerror = err => reject(err);
            reader.readAsDataURL(file);
        });
    }

    async function saveFileToDB(imgurURL) {
        images.push({
            date: new Date(),
            image: imgurURL,
        });
        const res = await fetch(DB, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(images),
        });
        console.log(images);
        imagesStore.update(imagesFromStore => (imagesFromStore = images));
        return res;
    }

    async function uploadFile(file) {
        try {
            if (skip_saving === false && file) {
                uploading = true;
                // const encedImage = await encodeImage(file);
                const form = new FormData();

                form.append('image', file);

                const imageUploadRes = await fetch(
                    'https://api.imgur.com/3/image',
                    {
                        method: 'POST',
                        headers: {
                            Authorization: `Client-ID ${clientID}`,
                            Accept: 'application/json',
                        },
                        body: form,
                    },
                );
                uploading = false;
                const payload = await imageUploadRes.json();
                debugger;
                await saveFileToDB(payload.data.link);
            } else {
                console.log(
                    'Flag "skip_saving" is enabled or empty file selected',
                );
                return true;
            }
        } catch (err) {
            console.error('Fail to upload new image or process image');
            console.error(err);
        }
    }

    async function uploadHandler(e) {
        const file = e.target.files[0];
        try {
            const res = await uploadFile(file);
        } catch (err) {
            console.error('Fail to upload new image');
            console.error(err);
        }
    }

    window.addEventListener('paste', e => {
        const items = (event.clipboardData || event.originalEvent.clipboardData)
            .items;
        for (const index in items) {
            var item = items[index];
            if (item.kind === 'file') {
                var blob = item.getAsFile();
                uploadFile(blob);
            }
        }
    });
</script>

<style>
    .upload_container {
        width: 100%;
        position: relative;
        display: flex;
        justify-content: center;
    }

    #uploadbox {
        width: 40em;
        height: 30em;
        border: 1px dotted #161616;
    }
</style>

{#if uploading}
    <p>Uploading</p>
{/if}

<div class="upload_container">
    <input
        type="file"
        accept="image/*"
        on:change={uploadHandler}
        id="uploadbox" />
</div>
