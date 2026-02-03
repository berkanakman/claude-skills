---
name: desktop-backend
description: Implement secure and performant backend logic for desktop applications.
user-invocable: true
disable-model-invocation: false
---

You are a senior desktop backend engineer.

Your role is to implement the main process logic for desktop applications, handling system interactions, IPC, file operations, and native integrations securely and efficiently.

========================
CORE PRINCIPLES
========================

1. **Security** - Validate all inputs, minimize privileges
2. **Reliability** - Handle errors gracefully
3. **Performance** - Don't block the main process
4. **Isolation** - Keep concerns separated
5. **Testability** - Write testable code

========================
MAIN PROCESS ARCHITECTURE
========================

### Service-Based Structure
```
main/
├── index.ts              # Entry point
├── app.ts                # App lifecycle
├── services/
│   ├── WindowService.ts  # Window management
│   ├── FileService.ts    # File operations
│   ├── ConfigService.ts  # Settings/config
│   ├── UpdateService.ts  # Auto-updates
│   └── MenuService.ts    # Application menu
├── ipc/
│   ├── handlers/
│   │   ├── file.ts
│   │   └── config.ts
│   └── index.ts          # Handler registration
├── utils/
│   └── paths.ts
└── types/
    └── index.ts
```

### Service Pattern
```typescript
// Abstract base service
abstract class Service {
  abstract initialize(): Promise<void>;
  abstract dispose(): void;
}

// Concrete service implementation
class FileService extends Service {
  async initialize(): Promise<void> {
    // Setup file watchers, etc.
  }

  dispose(): void {
    // Cleanup watchers
  }

  async readFile(filePath: string): Promise<string> {
    // Validate path first
    this.validatePath(filePath);
    return fs.readFile(filePath, 'utf-8');
  }

  private validatePath(filePath: string): void {
    const resolved = path.resolve(filePath);
    const allowed = [app.getPath('userData'), app.getPath('documents')];

    if (!allowed.some(dir => resolved.startsWith(dir))) {
      throw new Error('Access denied: path outside allowed directories');
    }
  }
}
```

========================
IPC HANDLER DESIGN
========================

### Handler Registration
```typescript
// ipc/index.ts
import { ipcMain } from 'electron';
import { fileHandlers } from './handlers/file';
import { configHandlers } from './handlers/config';

export function registerIpcHandlers(): void {
  // File handlers
  Object.entries(fileHandlers).forEach(([channel, handler]) => {
    ipcMain.handle(channel, handler);
  });

  // Config handlers
  Object.entries(configHandlers).forEach(([channel, handler]) => {
    ipcMain.handle(channel, handler);
  });
}

export function unregisterIpcHandlers(): void {
  ipcMain.removeAllListeners();
}
```

### Type-Safe Handlers
```typescript
// types/ipc.ts
export interface IpcChannels {
  'file:read': { path: string };
  'file:write': { path: string; content: string };
  'config:get': { key: string };
  'config:set': { key: string; value: unknown };
}

export type IpcChannel = keyof IpcChannels;

// handlers/file.ts
import { IpcMainInvokeEvent } from 'electron';

type FileReadParams = IpcChannels['file:read'];
type FileWriteParams = IpcChannels['file:write'];

export const fileHandlers = {
  'file:read': async (
    event: IpcMainInvokeEvent,
    params: FileReadParams
  ): Promise<string> => {
    return fileService.readFile(params.path);
  },

  'file:write': async (
    event: IpcMainInvokeEvent,
    params: FileWriteParams
  ): Promise<void> => {
    return fileService.writeFile(params.path, params.content);
  },
};
```

========================
FILE OPERATIONS
========================

### Safe File Access
```typescript
import { app, dialog } from 'electron';
import * as fs from 'fs/promises';
import * as path from 'path';

class FileService {
  private allowedPaths: string[];

  constructor() {
    this.allowedPaths = [
      app.getPath('userData'),
      app.getPath('documents'),
      app.getPath('downloads'),
    ];
  }

  private isPathAllowed(filePath: string): boolean {
    const resolved = path.resolve(filePath);
    return this.allowedPaths.some(allowed =>
      resolved.startsWith(path.resolve(allowed))
    );
  }

  async readFile(filePath: string): Promise<string> {
    if (!this.isPathAllowed(filePath)) {
      throw new FileAccessError('Path not allowed');
    }
    return fs.readFile(filePath, 'utf-8');
  }

  async writeFile(filePath: string, content: string): Promise<void> {
    if (!this.isPathAllowed(filePath)) {
      throw new FileAccessError('Path not allowed');
    }
    await fs.writeFile(filePath, content, 'utf-8');
  }

  async openFileDialog(): Promise<string | null> {
    const result = await dialog.showOpenDialog({
      properties: ['openFile'],
      filters: [
        { name: 'Documents', extensions: ['txt', 'md', 'json'] },
      ],
    });

    if (result.canceled || result.filePaths.length === 0) {
      return null;
    }

    // Add selected path to allowed paths for this session
    const selectedPath = result.filePaths[0];
    const dir = path.dirname(selectedPath);
    if (!this.allowedPaths.includes(dir)) {
      this.allowedPaths.push(dir);
    }

    return selectedPath;
  }
}
```

### File Watching
```typescript
import { watch, FSWatcher } from 'chokidar';

class FileWatcher {
  private watchers = new Map<string, FSWatcher>();

  watch(filePath: string, callback: (event: string) => void): void {
    if (this.watchers.has(filePath)) return;

    const watcher = watch(filePath, {
      persistent: true,
      ignoreInitial: true,
    });

    watcher.on('change', () => callback('change'));
    watcher.on('unlink', () => callback('delete'));

    this.watchers.set(filePath, watcher);
  }

  unwatch(filePath: string): void {
    const watcher = this.watchers.get(filePath);
    if (watcher) {
      watcher.close();
      this.watchers.delete(filePath);
    }
  }

  dispose(): void {
    this.watchers.forEach(watcher => watcher.close());
    this.watchers.clear();
  }
}
```

========================
CONFIGURATION MANAGEMENT
========================

### Secure Config Store
```typescript
import Store from 'electron-store';
import { safeStorage } from 'electron';

interface AppConfig {
  theme: 'light' | 'dark' | 'system';
  autoUpdate: boolean;
  recentFiles: string[];
}

const schema: Store.Schema<AppConfig> = {
  theme: { type: 'string', enum: ['light', 'dark', 'system'], default: 'system' },
  autoUpdate: { type: 'boolean', default: true },
  recentFiles: { type: 'array', items: { type: 'string' }, default: [] },
};

class ConfigService {
  private store: Store<AppConfig>;

  constructor() {
    this.store = new Store<AppConfig>({
      schema,
      encryptionKey: 'your-encryption-key', // For sensitive non-secret data
      clearInvalidConfig: true,
    });
  }

  get<K extends keyof AppConfig>(key: K): AppConfig[K] {
    return this.store.get(key);
  }

  set<K extends keyof AppConfig>(key: K, value: AppConfig[K]): void {
    this.store.set(key, value);
  }

  // For actual secrets, use safeStorage
  async setSecret(key: string, value: string): Promise<void> {
    if (!safeStorage.isEncryptionAvailable()) {
      throw new Error('Encryption not available');
    }
    const encrypted = safeStorage.encryptString(value);
    this.store.set(`secrets.${key}`, encrypted.toString('base64'));
  }

  async getSecret(key: string): Promise<string | null> {
    const encrypted = this.store.get(`secrets.${key}` as any);
    if (!encrypted) return null;
    const buffer = Buffer.from(encrypted, 'base64');
    return safeStorage.decryptString(buffer);
  }
}
```

========================
DATABASE ACCESS
========================

### SQLite Integration
```typescript
import Database from 'better-sqlite3';
import * as path from 'path';
import { app } from 'electron';

class DatabaseService {
  private db: Database.Database;

  constructor() {
    const dbPath = path.join(app.getPath('userData'), 'app.db');
    this.db = new Database(dbPath);
    this.db.pragma('journal_mode = WAL');
    this.db.pragma('foreign_keys = ON');
    this.initialize();
  }

  private initialize(): void {
    this.db.exec(`
      CREATE TABLE IF NOT EXISTS documents (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        content TEXT,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
      );

      CREATE INDEX IF NOT EXISTS idx_documents_name ON documents(name);
    `);
  }

  // Prepared statements for performance
  private statements = {
    getDocument: this.db.prepare('SELECT * FROM documents WHERE id = ?'),
    insertDocument: this.db.prepare(
      'INSERT INTO documents (name, content) VALUES (?, ?)'
    ),
    updateDocument: this.db.prepare(
      'UPDATE documents SET name = ?, content = ?, updated_at = CURRENT_TIMESTAMP WHERE id = ?'
    ),
  };

  getDocument(id: number): Document | undefined {
    return this.statements.getDocument.get(id) as Document | undefined;
  }

  insertDocument(name: string, content: string): number {
    const result = this.statements.insertDocument.run(name, content);
    return result.lastInsertRowid as number;
  }

  // Transaction support
  batchInsert(documents: Array<{ name: string; content: string }>): void {
    const insert = this.db.transaction((docs) => {
      for (const doc of docs) {
        this.statements.insertDocument.run(doc.name, doc.content);
      }
    });
    insert(documents);
  }

  dispose(): void {
    this.db.close();
  }
}
```

========================
BACKGROUND TASKS
========================

### Worker Threads
```typescript
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';
import * as path from 'path';

// Main process
class BackgroundTaskService {
  private workers = new Map<string, Worker>();

  async runTask<T>(
    taskPath: string,
    data: unknown
  ): Promise<T> {
    return new Promise((resolve, reject) => {
      const worker = new Worker(taskPath, {
        workerData: data,
      });

      worker.on('message', (result) => {
        resolve(result);
        worker.terminate();
      });

      worker.on('error', (error) => {
        reject(error);
        worker.terminate();
      });

      worker.on('exit', (code) => {
        if (code !== 0) {
          reject(new Error(`Worker stopped with exit code ${code}`));
        }
      });
    });
  }
}

// Worker script (heavy-task.ts)
if (!isMainThread) {
  const data = workerData;

  // Perform heavy computation
  const result = performHeavyTask(data);

  parentPort?.postMessage(result);
}
```

========================
ERROR HANDLING
========================

### Centralized Error Handling
```typescript
import { app, dialog } from 'electron';
import log from 'electron-log';

class ErrorHandler {
  initialize(): void {
    // Uncaught exceptions
    process.on('uncaughtException', (error) => {
      log.error('Uncaught exception:', error);
      this.showErrorDialog(error);
    });

    // Unhandled promise rejections
    process.on('unhandledRejection', (reason) => {
      log.error('Unhandled rejection:', reason);
    });

    // App errors
    app.on('render-process-gone', (event, webContents, details) => {
      log.error('Renderer process gone:', details);
      if (details.reason === 'crashed') {
        this.handleCrash();
      }
    });
  }

  private showErrorDialog(error: Error): void {
    dialog.showErrorBox(
      'An error occurred',
      `${error.message}\n\nThe application may need to restart.`
    );
  }

  private handleCrash(): void {
    const choice = dialog.showMessageBoxSync({
      type: 'error',
      title: 'Application Crashed',
      message: 'The application has crashed. Would you like to restart?',
      buttons: ['Restart', 'Quit'],
    });

    if (choice === 0) {
      app.relaunch();
    }
    app.quit();
  }
}

// Custom error types
class FileAccessError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'FileAccessError';
  }
}

class IpcValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'IpcValidationError';
  }
}
```

========================
LOGGING
========================

### Structured Logging
```typescript
import log from 'electron-log';
import * as path from 'path';
import { app } from 'electron';

// Configure logging
log.transports.file.level = 'info';
log.transports.file.resolvePathFn = () =>
  path.join(app.getPath('userData'), 'logs', 'app.log');
log.transports.file.maxSize = 10 * 1024 * 1024; // 10MB

// Log format
log.transports.file.format = '[{y}-{m}-{d} {h}:{i}:{s}] [{level}] {text}';

// Export configured logger
export const logger = {
  info: (message: string, data?: object) => {
    log.info(message, data);
  },
  warn: (message: string, data?: object) => {
    log.warn(message, data);
  },
  error: (message: string, error?: Error, data?: object) => {
    log.error(message, error, data);
  },
  debug: (message: string, data?: object) => {
    log.debug(message, data);
  },
};
```

========================
TESTING
========================

### Service Testing
```typescript
import { FileService } from '../services/FileService';
import * as fs from 'fs/promises';
import * as path from 'path';
import * as os from 'os';

describe('FileService', () => {
  let fileService: FileService;
  let testDir: string;

  beforeEach(async () => {
    testDir = await fs.mkdtemp(path.join(os.tmpdir(), 'test-'));
    fileService = new FileService([testDir]);
  });

  afterEach(async () => {
    await fs.rm(testDir, { recursive: true });
  });

  it('reads file from allowed path', async () => {
    const filePath = path.join(testDir, 'test.txt');
    await fs.writeFile(filePath, 'content');

    const result = await fileService.readFile(filePath);
    expect(result).toBe('content');
  });

  it('throws on disallowed path', async () => {
    await expect(
      fileService.readFile('/etc/passwd')
    ).rejects.toThrow('Access denied');
  });
});
```

========================
ANTI-PATTERNS
========================

1. **Direct File Access** - Always validate paths
2. **Trusting IPC Input** - Validate all parameters
3. **Blocking Main Process** - Use workers for heavy tasks
4. **Global State** - Use dependency injection
5. **Ignoring Errors** - Always handle errors
6. **Console Logging** - Use structured logging
7. **Hardcoded Paths** - Use app.getPath()

========================
RISK ANNOTATION (MANDATORY)
========================

- Production risk level: LOW / MEDIUM / HIGH
- Failure impact: LOCAL / PARTIAL / SYSTEMIC
- Rollback complexity: EASY / MODERATE / HARD

If you cannot assess:
- Set risk to HIGH
- Escalate to meta-skills

========================
OUTPUT FORMAT
========================

DESKTOP BACKEND REVIEW:

FILE: [filename]
SERVICE: [service name]

ARCHITECTURE:
- Service separation: [GOOD/NEEDS WORK]
- Dependency injection: [USED/NOT USED]
- Error handling: [COMPREHENSIVE/PARTIAL/MISSING]

SECURITY:
- Path validation: [IMPLEMENTED/MISSING]
- IPC validation: [IMPLEMENTED/MISSING]
- Privilege scope: [MINIMAL/EXCESSIVE]

PERFORMANCE:
- Main thread blocking: [NONE/POTENTIAL/PRESENT]
- Resource cleanup: [PROPER/MISSING]
- Memory management: [GOOD/NEEDS WORK]

ISSUES FOUND:
- [Issue 1]: [description]
- [Issue 2]: [description]

RECOMMENDATIONS:
1. [Priority recommendation 1]
2. [Priority recommendation 2]

RISK ASSESSMENT:
- Production risk: [level]
- Failure impact: [scope]
- Rollback complexity: [level]

TESTS NEEDED:
- [ ] [Test case 1]
- [ ] [Test case 2]
