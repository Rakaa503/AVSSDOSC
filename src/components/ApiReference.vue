<script setup lang="ts">
import { computed, ref } from 'vue'
import CodeBlock from './CodeBlock.vue'

interface ApiEntry {
  name: string
  signature: string
  description: string
  example?: string
  note?: string
}

interface ApiSection {
  id: string
  title: string
  description: string
  entries: ApiEntry[]
}

interface ApiGroup {
  id: string
  label: string
  summary: string
  sections: ApiSection[]
}

const groups: ApiGroup[] = [
  {
    id: 'package',
    label: 'Package',
    summary: 'Package metadata, installation, and public entry points.',
    sections: [{
      id: 'installation',
      title: 'Installation',
      description: 'Install AVSS Lite and import its public entry points.',
      entries: [
        {
          name: 'Install package',
          signature: 'bun add github:Rakaa503/Avidev-Security',
          description: 'Installs AVSS Lite and its runtime dependencies.',
          example: 'bun add github:Rakaa503/Avidev-Security',
        },
        {
          name: 'Root entry point',
          signature: "import { validate, logger, createSecurity } from 'avss-lite-Raka503'",
          description: 'Exports CORS, environment, Prisma, validation, audit, doctor, errors, events, logger, plugins, security, and terminal modules.',
          example: "import { validate, logger, createSecurity } from 'avss-lite-Raka503'",
        },
        {
          name: 'Security entry point',
          signature: "import { createSecurity } from 'avss-lite-Raka503/security'",
          description: 'Dedicated package subpath for security-related exports.',
          example: "import { createSecurity, generateCsrfToken } from 'avss-lite-Raka503/security'",
        },
      ],
    }],
  },
  {
    id: 'core',
    label: 'Core Utilities',
    summary: 'Validation, environment parsing, Prisma, and CORS.',
    sections: [
      {
        id: 'validation',
        title: 'Validation',
        description: 'Zod-based input validation helpers.',
        entries: [
          {
            name: 'validate',
            signature: 'validate<T>(schema: ZodType<T>, input: unknown): ValidationResult<T>',
            description: 'Uses safeParse. Returns parsed data on success or issue messages on failure.',
            example: 'const result = validate(userSchema, requestBody)\n\nif (!result.success) {\n  console.error(result.errors)\n} else {\n  console.log(result.data)\n}',
          },
          {
            name: 'validateOrThrow',
            signature: 'validateOrThrow<T>(schema: ZodType<T>, input: unknown): T',
            description: 'Uses Zod parse and returns parsed data. Throws when validation fails.',
            example: 'const user = validateOrThrow(userSchema, requestBody)',
          },
          {
            name: 'ValidationResult<T>',
            signature: 'interface ValidationResult<T> { success: boolean; data?: T; errors?: string[] }',
            description: 'Result shape returned by validate.',
          },
        ],
      },
      {
        id: 'environment',
        title: 'Environment',
        description: 'Validates environment variables using the package Zod schema.',
        entries: [
          {
            name: 'validateEnv',
            signature: 'validateEnv(): Env',
            description: 'Parses process.env and returns validated environment values.',
            example: "import { validateEnv } from 'avss-lite-Raka503'\n\nconst env = validateEnv()\nconsole.log(env.NODE_ENV)",
            note: 'The current schema requires DATABASE_URL, BETTER_AUTH_SECRET (minimum 32 characters), and BETTER_AUTH_URL (valid URL). NODE_ENV defaults to development.',
          },
        ],
      },
      {
        id: 'prisma',
        title: 'Prisma',
        description: 'Exports a shared Prisma Client instance.',
        entries: [
          {
            name: 'prisma',
            signature: 'const prisma: PrismaClient',
            description: 'Exports a Prisma Client instance. In non-production environments it is cached on globalThis to reduce duplicate clients during reloads.',
            example: "import { prisma } from 'avss-lite-Raka503'\n\nconst users = await prisma.user.findMany()",
            note: 'Available delegates depend on the consumer project Prisma schema and generated client.',
          },
        ],
      },
      {
        id: 'cors',
        title: 'CORS',
        description: 'CORS middleware based on the cors package.',
        entries: [
          {
            name: 'corsMiddleware',
            signature: 'corsMiddleware',
            description: 'The current configuration reflects the request origin and enables credentials.',
            example: "import express from 'express'\nimport { corsMiddleware } from 'avss-lite-Raka503'\n\nconst app = express()\napp.use(corsMiddleware)",
            note: 'origin: true is not an explicit origin allowlist. Configure a restrictive origin policy for deployment.',
          },
        ],
      },
    ],
  },
  {
    id: 'operations',
    label: 'Operations',
    summary: 'Audit, health reporting, errors, events, logging, and plugins.',
    sections: [
      {
        id: 'audit',
        title: 'Audit',
        description: 'Console-oriented audit event helpers.',
        entries: [
          {
            name: 'audit',
            signature: 'const audit: Audit',
            description: 'Singleton providing log and convenience methods for account events.',
            example: "import { audit } from 'avss-lite-Raka503'\n\naudit.login({ id: 'user-01', name: 'Example User', email: 'user@example.com' })",
            note: 'The current implementation writes formatted entries to console. It does not persist audit records to a database or file.',
          },
          {
            name: 'Audit methods',
            signature: 'log · login · logout · register · failedLogin · changePassword · changeRole · createUser · updateUser · deleteUser · verifyEmail · resetPassword',
            description: 'Convenience methods create audit entries for corresponding events.',
          },
        ],
      },
      {
        id: 'doctor',
        title: 'Doctor',
        description: 'Collects caller-provided check results and renders a terminal report.',
        entries: [
          {
            name: 'doctor.check',
            signature: 'check(name: string, success: boolean): void',
            description: 'Adds a named check result. The caller must execute the check and provide its result.',
            example: "import { doctor } from 'avss-lite-Raka503'\n\ndoctor.check('Database connection', databaseConnected)\ndoctor.render()",
          },
          {
            name: 'doctor.render',
            signature: 'render(): void',
            description: 'Renders collected PASS/FAIL results and a summary.',
            note: 'Doctor does not independently probe databases, services, or infrastructure.',
          },
        ],
      },
      {
        id: 'errors',
        title: 'Errors',
        description: 'Structured application errors and normalized error responses.',
        entries: [
          {
            name: 'AVSSError',
            signature: 'new AVSSError(message: string, status: number, code: ErrorCode)',
            description: 'Base error class with status, code, timestamp, and toJSON().',
          },
          {
            name: 'errorHandler',
            signature: 'errorHandler(error: unknown): ErrorResponse',
            description: 'Converts AVSSError instances into the package error response. Unknown errors become a generic 500 response.',
            example: "import { errorHandler, UnauthorizedError } from 'avss-lite-Raka503'\n\nconst response = errorHandler(new UnauthorizedError())",
          },
          {
            name: 'Error subclasses',
            signature: 'UnauthorizedError · ForbiddenError · ValidationError · SessionExpiredError · InternalServerError',
            description: 'Predefined errors map to HTTP statuses 401, 403, 400, 401, and 500 respectively.',
          },
        ],
      },
      {
        id: 'events',
        title: 'Events',
        description: 'Synchronous in-process event emitter.',
        entries: [
          {
            name: 'events',
            signature: 'const events: EventEmitter',
            description: 'Singleton supporting on, off, emit, clear, and listenerCount.',
            example: "import { events } from 'avss-lite-Raka503'\n\nevents.on('login', (payload) => {\n  console.log(payload.name)\n})\n\nevents.emit('login', { id: 'u1', name: 'Example User' })",
          },
        ],
      },
      {
        id: 'logger',
        title: 'Logger',
        description: 'Console and file logging utilities.',
        entries: [
          {
            name: 'logger',
            signature: 'const logger: Logger',
            description: 'Exposes info, success, warn, error, debug, security, database, access, and fatal methods.',
            example: "import { logger } from 'avss-lite-Raka503'\n\nlogger.info('Application started')\nlogger.security('Authentication attempt')\nlogger.error('Unexpected failure')",
          },
          {
            name: 'logRequest',
            signature: 'logRequest(request: { method: string; url: string; status: number; ip?: string }): void',
            description: 'Writes a formatted ACCESS log entry for a request.',
          },
          {
            name: 'Timer utilities',
            signature: 'startTimer(name: string): void · endTimer(name: string): void',
            description: 'Measures elapsed time between a named timer start and end.',
          },
          {
            name: 'rotateLogs',
            signature: 'rotateLogs(): void',
            description: 'Checks whether the log directory exists and prints a rotation-check message.',
            note: 'The current implementation does not rotate, archive, or delete log files.',
          },
        ],
      },
      {
        id: 'plugins',
        title: 'Plugins',
        description: 'Minimal plugin interface and built-in plugin factories.',
        entries: [
          {
            name: 'AVSSPlugin',
            signature: 'interface AVSSPlugin { name: string; setup(): void }',
            description: 'Defines the plugin contract.',
          },
          {
            name: 'Built-in plugins',
            signature: 'loggerPlugin(): AVSSPlugin · auditPlugin(): AVSSPlugin',
            description: 'Factories return plugins whose setup methods print a loaded message.',
            note: 'There is no automatic plugin registry or lifecycle manager in the current implementation.',
          },
        ],
      },
    ],
  },
  {
    id: 'security',
    label: 'Security Framework',
    summary: 'Security facade, CSRF, XSS, headers, cookies, proxy, and rate limiting.',
    sections: [
      {
        id: 'security-overview',
        title: 'Security Framework',
        description: 'Create a facade that groups security helper functions.',
        entries: [
          {
            name: 'createSecurity',
            signature: 'createSecurity(config?: SecurityConfig): SecurityFramework',
            description: 'Creates a SecurityFramework instance and applies configuration to the shared security configuration.',
            example: "import { createSecurity } from 'avss-lite-Raka503/security'\n\nconst security = createSecurity()\nconst token = security.csrf.generateToken()",
          },
          {
            name: 'SecurityConfig',
            signature: 'csrf? · xss? · headers? · cors? · cookies? · proxy? · rateLimit?',
            description: 'Accepts partial configuration objects for supported security modules.',
          },
          {
            name: 'Configuration helpers',
            signature: 'setSecurityConfig(value) · getSecurityConfig() · resetSecurityConfig()',
            description: 'Updates, reads a frozen shallow copy of, or resets the shared security configuration.',
          },
        ],
      },
      {
        id: 'csrf',
        title: 'CSRF',
        description: 'Token generation and request-token comparison.',
        entries: [
          {
            name: 'generateCsrfToken',
            signature: 'generateCsrfToken(config?: CsrfConfig): string',
            description: 'Generates a hexadecimal token using Node.js randomBytes. The default configuration requests 32 random bytes, producing a 64-character hexadecimal token.',
          },
          {
            name: 'validateCsrfToken',
            signature: 'validateCsrfToken(token: string, expected: string): boolean',
            description: 'Returns true when a non-empty token exactly matches the expected token.',
            note: 'The current comparison uses ordinary string equality, not a constant-time comparison.',
          },
          {
            name: 'csrfMiddleware',
            signature: 'csrfMiddleware(request: CsrfRequest, config?: CsrfConfig): CsrfResult',
            description: 'Checks a configured CSRF header in a plain request-shaped object. It is not an Express or Fastify middleware handler.',
          },
        ],
      },
      {
        id: 'xss',
        title: 'XSS',
        description: 'String trimming and HTML character escaping.',
        entries: [
          {
            name: 'sanitize',
            signature: 'sanitize(input: string, config?: XssConfig): string',
            description: 'Applies configured trimming and HTML escaping. Both are enabled by default.',
            example: "const safeText = security.xss.sanitize('<script>alert(1)<\/script>')",
            note: 'This is HTML escaping, not a full parser-based sanitizer. It does not validate URL schemes such as javascript:. Use context-appropriate output encoding.',
          },
          {
            name: 'escapeHtml',
            signature: 'escapeHtml(value: string): string',
            description: 'Escapes ampersand, angle brackets, quotes, apostrophes, and forward slash.',
          },
        ],
      },
      {
        id: 'headers',
        title: 'Security Headers',
        description: 'Builds a map of default security response header values.',
        entries: [
          {
            name: 'SecurityHeadersBuilder',
            signature: 'create() · withDefaults(config?) · add(name, value) · remove(name) · build()',
            description: 'Provides a chainable builder that returns a frozen header map.',
            example: "const headers = security.headers.builder()\n  .add('X-App-Version', '1.0.0')\n  .build()",
          },
          {
            name: 'Default headers',
            signature: 'Content-Security-Policy · Strict-Transport-Security · X-Frame-Options · X-Content-Type-Options · Referrer-Policy · Permissions-Policy',
            description: 'Defaults include CSP, HSTS, DENY framing, nosniff, strict-origin-when-cross-origin, and a restrictive permissions policy.',
            note: 'The builder returns header values; it does not attach them to an HTTP response. HSTS should only be used over HTTPS. Review CSP for application-specific requirements.',
          },
        ],
      },
      {
        id: 'cookies',
        title: 'Cookies',
        description: 'Cookie header parsing and serialization helpers.',
        entries: [
          {
            name: 'parseCookies',
            signature: 'parseCookies(cookieHeader: string): CookieMap',
            description: 'Parses a Cookie header into a name/value object. Later duplicate names overwrite earlier ones.',
          },
          {
            name: 'serializeCookie',
            signature: 'serializeCookie(name: string, value: string, config?: CookieConfig): string',
            description: 'Serializes a cookie with Path, Max-Age, SameSite, and configured HttpOnly and Secure attributes.',
            example: "const cookie = security.cookies.serialize('avss.session', sessionId)",
            note: 'Returns a Set-Cookie value; it does not write the HTTP response. Invalid percent-encoding during parsing may throw.',
          },
        ],
      },
      {
        id: 'proxy',
        title: 'Proxy',
        description: 'Reads proxy-related request headers and checks configured proxy addresses.',
        entries: [
          {
            name: 'getProxyInfo',
            signature: 'getProxyInfo(headers: ProxyHeaders, config?: ProxyConfig): ProxyInfo',
            description: 'Returns a client IP and optionally forwarded host and protocol values.',
          },
          {
            name: 'isTrustedProxy',
            signature: 'isTrustedProxy(ip: string, config?: ProxyConfig): boolean',
            description: 'Checks whether an IP exactly matches an entry in trustedProxies.',
            note: 'getProxyInfo does not verify the sender against trustedProxies before reading forwarded headers. Validate the immediate proxy before trusting forwarded headers.',
          },
        ],
      },
      {
        id: 'rate-limit',
        title: 'Rate Limit',
        description: 'In-memory request counters keyed by caller-supplied identifiers.',
        entries: [
          {
            name: 'rateLimit',
            signature: 'rateLimit(key: string, config?: RateLimitConfig): RateLimitResult',
            description: 'Increments the counter for a key and returns success, remaining, and retryAfter values.',
            example: "const result = security.rateLimit.check('client-ip')\n\nif (!result.success) {\n  console.warn('Retry after ' + result.retryAfter + ' seconds')\n}",
            note: 'The implementation is in-memory and process-local. It is not HTTP middleware and is not shared across application instances.',
          },
          {
            name: 'RateLimiter',
            signature: 'new RateLimiter(options: RateLimiterOptions)',
            description: 'A separate class-based limiter with consume, stats, keys, and clear methods, backed by its own MemoryStore.',
          },
        ],
      },
    ],
  },
  {
    id: 'terminal',
    label: 'Terminal',
    summary: 'Terminal formatting, status output, spinner, progress, timer, and tree helpers.',
    sections: [{
      id: 'terminal-utilities',
      title: 'Terminal Utilities',
      description: 'Convenience helpers for command-line output.',
      entries: [
        {
          name: 'Status output',
          signature: 'success(message) · error(message) · warning(message) · info(message) · pass(message) · fail(message)',
          description: 'Prints status messages. pass delegates to success and fail delegates to error.',
        },
        {
          name: 'Spinner',
          signature: 'new Spinner() · start() · success(text?) · fail(text?) · warn(text?) · info(text?) · stop()',
          description: 'Wraps ora to display a terminal spinner and completion states.',
        },
        {
          name: 'Timer',
          signature: 'new Timer() · reset(): void · elapsed(): number',
          description: 'Measures elapsed time in milliseconds using Date.now().',
        },
        {
          name: 'Progress',
          signature: 'progress(current?: number, total?: number, width?: number): void',
          description: 'Prints a text progress indicator. Defaults to current 1, total 1, and width 30.',
          note: 'The implementation does not validate zero, negative, or out-of-range values.',
        },
        {
          name: 'Layout helpers',
          signature: 'banner() · box(title, width?) · divider(char?, length?) · header(title) · section(title) · footer(options) · tree(title, items)',
          description: 'Prints terminal banners, boxes, dividers, headings, sections, footers, and a flat list.',
          note: 'The footer currently prints READY FOR PRODUCTION regardless of the supplied status value.',
        },
        {
          name: 'table',
          signature: 'table(): void',
          description: 'Currently prints an empty console table; it does not accept row data.',
        },
      ],
    }],
  },
]

const activeGroup = ref('package')
const query = ref('')

const currentGroup = computed(
  () => groups.find((group) => group.id === activeGroup.value) ?? groups[0],
)

const filteredSections = computed(() => {
  const value = query.value.trim().toLowerCase()
  if (!value) return currentGroup.value.sections

  return currentGroup.value.sections
    .map((section) => ({
      ...section,
      entries: section.entries.filter((entry) =>
        (entry.name + ' ' + entry.signature + ' ' + entry.description + ' ' + (entry.note ?? '')).toLowerCase().includes(value),
      ),
    }))
    .filter((section) =>
      section.title.toLowerCase().includes(value) ||
      section.description.toLowerCase().includes(value) ||
      section.entries.length > 0,
    )
})

function selectGroup(id: string) {
  activeGroup.value = id
  query.value = ''
}

function getCodeLanguage(entry: ApiEntry): string {
  if (/^(npm|npx|pnpm|yarn|bun)\s/.test(entry.example ?? '')) {
    return 'bash'
  }

  return 'typescript'
}

function getCodeFilename(entry: ApiEntry): string {
  const slug = entry.name
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/^-|-$/g, '')

  const extension = getCodeLanguage(entry) === 'bash' ? 'sh' : 'ts'

  return `${slug || 'example'}.${extension}`
}
</script>

<template>
  <div class="api-reference">
    <div class="api-intro">
      <div class="api-kicker">PUBLIC API / REFERENCE</div>
      <p>Explore the public modules, function signatures, configuration types, and implementation notes available in AVSS Lite.</p>
      <div class="api-meta">
        <span class="api-version">v1.0.2</span>
        <span>TypeScript</span>
        <span>ESM / CommonJS</span>
      </div>
    </div>

    <div class="api-browser">
      <aside class="api-nav" aria-label="API categories">
        <span class="api-nav-label">CATEGORIES</span>
        <button
          v-for="group in groups"
          :key="group.id"
          type="button"
          class="api-nav-item"
          :class="{ active: activeGroup === group.id }"
          @click="selectGroup(group.id)"
        >
          <span>{{ group.label }}</span>
          <span class="api-nav-arrow">↗</span>
        </button>
      </aside>

      <div class="api-content">
        <div class="api-content-heading">
          <div>
            <span class="api-overline">MODULE REFERENCE</span>
            <h2>{{ currentGroup.label }}</h2>
            <p>{{ currentGroup.summary }}</p>
          </div>
        </div>

        <label class="api-search">
          <span aria-hidden="true">⌕</span>
          <input v-model="query" type="search" placeholder="Filter this category..." aria-label="Filter API entries" />
        </label>

        <div v-if="filteredSections.length === 0" class="api-empty">
          No API entries match "{{ query }}".
        </div>

        <section v-for="section in filteredSections" :id="section.id" :key="section.id" class="api-section">
          <div class="api-section-heading">
            <span class="api-section-mark"></span>
            <h3>{{ section.title }}</h3>
          </div>
          <p class="api-section-description">{{ section.description }}</p>

          <article v-for="entry in section.entries" :key="entry.name" class="api-entry">
  <template v-if="section.id === 'installation' && entry.name === 'Install package'">
    <div class="install-card">
      <div class="install-card-header">
        <div>
          <span class="install-eyebrow">INSTALLATION METHOD</span>
          <h4>Install AVSS Lite</h4>
          <p>Install directly from the official AVSS Lite GitHub repository using Bun.</p>
        </div>
        <span class="install-badge">BUN</span>
      </div>

      <div class="install-repository">
        <span>OFFICIAL GITHUB REPOSITORY</span>
        <a href="https://github.com/Rakaa503/Avidev-Security" target="_blank" rel="noopener noreferrer">
          github.com/Rakaa503/Avidev-Security ↗
        </a>
      </div>

      <div class="api-code-wrap">
        <div class="api-code-heading">
          <span>TERMINAL</span>
          <span>INSTALL COMMAND</span>
        </div>
        <CodeBlock
          :code="entry.example ?? ''"
          language="bash"
          filename="terminal.sh"
        />
      </div>

      <div class="install-note">
        <span>i</span>
        <p>AVSS Lite is distributed through GitHub. Bun installs the package directly from its repository, without requiring a separate git clone.</p>
      </div>
    </div>
  </template>

  <template v-else>
    <div class="api-entry-heading">
      <h4>{{ entry.name }}</h4>
      <span class="api-entry-type">API</span>
    </div>
    <pre class="api-signature"><code>{{ entry.signature }}</code></pre>
    <p class="api-entry-description">{{ entry.description }}</p>

    <div v-if="entry.example" class="api-code-wrap">
      <div class="api-code-heading">
        <span>EXAMPLE</span>
      </div>
      <CodeBlock
        :code="entry.example"
        :language="getCodeLanguage(entry)"
        :filename="getCodeFilename(entry)"
      />
    </div>

    <div v-if="entry.note" class="api-note">
      <span>NOTE</span>
      <p>{{ entry.note }}</p>
    </div>
  </template>
</article>
        </section>

        <div class="api-reference-footer">
          <span>AVSS LITE · API REFERENCE</span>
          <span>VERIFY AGAINST YOUR INSTALLED VERSION</span>
        </div>
      </div>
    </div>
  </div>
</template>


