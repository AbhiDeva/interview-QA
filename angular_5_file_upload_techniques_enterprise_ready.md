# Angular – 5 File Upload Techniques (Enterprise Ready)

This document covers **5 production‑ready Angular file upload techniques** with **detailed code**, **use cases**, and **enterprise considerations**.

---

## 1️⃣ Basic File Upload using HttpClient

### ✅ Use Case
- Small files
- Simple forms
- No progress tracking

### 📁 Component

```ts
@Component({
  selector: 'app-basic-upload',
  template: `
    <input type="file" (change)="onFileSelect($event)" />
    <button (click)="upload()">Upload</button>
  `
})
export class BasicUploadComponent {
  selectedFile!: File;

  constructor(private http: HttpClient) {}

  onFileSelect(event: Event) {
    const input = event.target as HTMLInputElement;
    if (input.files?.length) {
      this.selectedFile = input.files[0];
    }
  }

  upload() {
    const formData = new FormData();
    formData.append('file', this.selectedFile);

    this.http.post('/api/upload', formData).subscribe();
  }
}
```

### 🏢 Enterprise Notes
- No retry / progress
- Avoid for large files

---

## 2️⃣ File Upload with Progress Tracking

### ✅ Use Case
- UX feedback required
- Medium to large files

### 📁 Component

```ts
@Component({ selector: 'app-progress-upload' })
export class ProgressUploadComponent {
  progress = 0;

  constructor(private http: HttpClient) {}

  upload(file: File) {
    const formData = new FormData();
    formData.append('file', file);

    this.http.post('/api/upload', formData, {
      reportProgress: true,
      observe: 'events'
    }).subscribe(event => {
      if (event.type === HttpEventType.UploadProgress && event.total) {
        this.progress = Math.round(100 * event.loaded / event.total);
      }
    });
  }
}
```

### 🏢 Enterprise Notes
- Required for dashboards
- Combine with loader interceptor

---

## 3️⃣ Drag & Drop File Upload

### ✅ Use Case
- Modern UI
- Multiple file support

### 📁 Template

```html
<div class="drop-zone"
     (dragover)="$event.preventDefault()"
     (drop)="onDrop($event)">
  Drop files here
</div>
```

### 📁 Component

```ts
@Component({ selector: 'app-drag-upload' })
export class DragUploadComponent {
  constructor(private http: HttpClient) {}

  onDrop(event: DragEvent) {
    event.preventDefault();
    const files = event.dataTransfer?.files;
    if (files) {
      Array.from(files).forEach(file => this.upload(file));
    }
  }

  upload(file: File) {
    const fd = new FormData();
    fd.append('file', file);
    this.http.post('/api/upload', fd).subscribe();
  }
}
```

### 🏢 Enterprise Notes
- Combine with validation
- Use directive for reusability

---

## 4️⃣ Chunked File Upload (Large Files)

### ✅ Use Case
- Files > 100MB
- Unstable networks

### 📁 Component Logic

```ts
const CHUNK_SIZE = 5 * 1024 * 1024; // 5MB

uploadInChunks(file: File) {
  let start = 0;
  let index = 0;

  while (start < file.size) {
    const chunk = file.slice(start, start + CHUNK_SIZE);
    const fd = new FormData();

    fd.append('file', chunk);
    fd.append('index', index.toString());
    fd.append('fileName', file.name);

    this.http.post('/api/upload/chunk', fd).subscribe();

    start += CHUNK_SIZE;
    index++;
  }
}
```

### 🏢 Enterprise Notes
- Backend must merge chunks
- Supports resume upload

---

## 5️⃣ Reactive Form + Validation Upload

### ✅ Use Case
- Business rules
- File type / size validation

### 📁 Component

```ts
this.form = new FormGroup({
  file: new FormControl<File | null>(null, [Validators.required])
});
```

```ts
onFileChange(event: Event) {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (file && file.size < 5_000_000) {
    this.form.patchValue({ file });
  }
}
```

### 📁 Upload

```ts
upload() {
  const fd = new FormData();
  fd.append('file', this.form.value.file!);
  this.http.post('/api/upload', fd).subscribe();
}
```

### 🏢 Enterprise Notes
- Centralize validation
- Reusable form controls

---

## 🧠 Enterprise Recommendations

| Scenario | Technique |
|--------|-----------|
| Profile image | Basic upload |
| Reports | Progress upload |
| Media apps | Drag & drop |
| Huge files | Chunk upload |
| Business forms | Reactive form upload |

---

## ✅ Next Enhancements
- Retry & resume uploads
- Virus scan integration
- AWS S3 pre‑signed uploads
- Upload queue with RxJS
- File upload reusable library

---

If you want:
- **Interview questions on file upload**
- **AWS / Azure / GCP upload patterns**
- **Secure enterprise upload architecture**

Tell me 👍

