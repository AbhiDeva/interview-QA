# Angular – Multiple File Upload with Progress (Chunked, Large Files)

This guide shows a **production-grade Angular implementation** for uploading **multiple large files** using **chunking**, with **per-file and overall progress tracking**.

---

## 🎯 Use Case
- Upload files **>100MB**
- Multiple files in parallel or sequence
- Show **individual progress bars**
- Resume / retry capable backend

---

## 🧱 Architecture Overview

- **Component** → UI & progress rendering
- **UploadService** → Chunk logic & HTTP
- **Backend API** → Accepts chunks & merges

```
UI → UploadService → /upload/chunk → Storage
```

---

## 📁 Constants

```ts
export const CHUNK_SIZE = 10 * 1024 * 1024; // 10MB
```

---

## 📁 Upload Model

```ts
export interface UploadFile {
  file: File;
  progress: number;
  uploadedBytes: number;
  status: 'pending' | 'uploading' | 'completed' | 'error';
}
```

---

## 🛠 Upload Service (Chunk + Progress)

```ts
@Injectable({ providedIn: 'root' })
export class ChunkUploadService {

  constructor(private http: HttpClient) {}

  uploadFile(file: File, onProgress: (p: number) => void): Observable<void> {
    const totalChunks = Math.ceil(file.size / CHUNK_SIZE);
    let currentChunk = 0;

    return new Observable(observer => {
      const uploadNext = () => {
        const start = currentChunk * CHUNK_SIZE;
        const end = Math.min(start + CHUNK_SIZE, file.size);
        const chunk = file.slice(start, end);

        const formData = new FormData();
        formData.append('chunk', chunk);
        formData.append('fileName', file.name);
        formData.append('chunkIndex', currentChunk.toString());
        formData.append('totalChunks', totalChunks.toString());

        this.http.post('/api/upload/chunk', formData, {
          reportProgress: true,
          observe: 'events'
        }).subscribe({
          next: event => {
            if (event.type === HttpEventType.UploadProgress && event.total) {
              const chunkProgress = event.loaded / event.total;
              const overallProgress = Math.round(
                ((currentChunk + chunkProgress) / totalChunks) * 100
              );
              onProgress(overallProgress);
            }

            if (event.type === HttpEventType.Response) {
              currentChunk++;
              currentChunk < totalChunks ? uploadNext() : observer.complete();
            }
          },
          error: err => observer.error(err)
        });
      };

      uploadNext();
    });
  }
}
```

---

## 📁 Component – Multiple File Upload

```ts
@Component({
  selector: 'app-multi-upload',
  templateUrl: './multi-upload.component.html'
})
export class MultiUploadComponent {

  uploads: UploadFile[] = [];

  constructor(private uploadService: ChunkUploadService) {}

  onFilesSelected(event: Event) {
    const files = (event.target as HTMLInputElement).files;
    if (!files) return;

    this.uploads = Array.from(files).map(file => ({
      file,
      progress: 0,
      uploadedBytes: 0,
      status: 'pending'
    }));
  }

  startUpload() {
    this.uploads.forEach(upload => {
      upload.status = 'uploading';

      this.uploadService.uploadFile(upload.file, progress => {
        upload.progress = progress;
      }).subscribe({
        complete: () => upload.status = 'completed',
        error: () => upload.status = 'error'
      });
    });
  }
}
```

---

## 📁 Template – Progress UI

```html
<input type="file" multiple (change)="onFilesSelected($event)" />
<button (click)="startUpload()">Upload</button>

<div *ngFor="let u of uploads">
  <p>{{ u.file.name }} - {{ u.progress }}%</p>
  <progress [value]="u.progress" max="100"></progress>
  <span>{{ u.status }}</span>
</div>
```

---

## ⚡ Parallel vs Sequential Uploads

### Sequential (safe for backend)
```ts
from(this.uploads).pipe(
  concatMap(u => this.uploadService.uploadFile(u.file, p => u.progress = p))
)
```

### Parallel (faster, heavy load)
```ts
merge(...uploads.map(u => uploadFile(u)))
```

---

## 🧠 Enterprise Considerations

- Resume upload via `chunkIndex`
- Retry failed chunks only
- Backend validation (checksum)
- Virus scanning after merge
- Auth via interceptor

---

## 🧪 Backend Expectation (Pseudo)

```ts
POST /upload/chunk
- fileName
- chunkIndex
- totalChunks
- chunk
```

When `chunkIndex === totalChunks - 1` → merge chunks

---

## ✅ When to Use This Pattern

| Scenario | Recommendation |
|-------|----------------|
| Large media files | ✅ Required |
| Enterprise portals | ✅ Best practice |
| Mobile networks | ✅ Reliable |
| Small files | ❌ Overkill |

---

## 🚀 Enhancements
- Pause / Resume
- Upload queue
- Overall progress bar
- AWS S3 multipart upload
- Web Workers for slicing

---

If you want next:
- **Resume + retry implementation**
- **S3 multipart Angular version**
- **Interview questions (Senior level)**
- **Full-stack (Angular + Node) example**

Tell me 👍

