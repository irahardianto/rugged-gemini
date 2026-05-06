---
paths:
  - "**/app/**/*.rb"
  - "**/config/routes.rb"
  - "**/Gemfile"
---

## Rails Idioms and Patterns

Rails rewards convention over configuration, Active Record, and RESTful design. Idiomatic Rails = conventional, tested, Hotwire-aware.

> Scope: Rails-specific patterns. For Ruby: `@.gemini/skills/ruby-idioms/SKILL.md`.

### Active Record

1. **Scopes for reusable queries:**
   ```ruby
   class Task < ApplicationRecord
     scope :active, -> { where(status: :active) }
     scope :by_priority, -> { order(priority: :desc) }
   end
   ```

2. **Validations in models, not controllers:**
   ```ruby
   class Task < ApplicationRecord
     validates :title, presence: true, length: { maximum: 200 }
     validates :priority, inclusion: { in: %w[low medium high] }
   end
   ```

3. **`includes`/`preload`** for eager loading — avoid N+1.

### Controllers

1. **RESTful actions** — only standard 7 actions per controller. Custom actions = new controller.
2. **Strong parameters** — never mass-assign without permit.
3. **Service objects** for complex business logic.

### Hotwire (7+)

1. **Turbo Frames** for partial page updates.
2. **Turbo Streams** for real-time updates.
3. **Stimulus** for JavaScript sprinkles — minimal JS.

### Testing

1. **RSpec (preferred):**
   ```ruby
   RSpec.describe TasksController, type: :request do
     describe 'POST /tasks' do
       it 'creates a task' do
         post tasks_path, params: { task: { title: 'Test', priority: 'high' } }
         expect(response).to have_http_status(:created)
       end
     end
   end
   ```

2. **FactoryBot** for test data. **Database Cleaner** for isolation.

### Formatting and Static Analysis

| Tool | Purpose | Command |
|---|---|---|
| RuboCop + rubocop-rails | Linting | `rubocop --autocorrect` |
| Brakeman | Security | `brakeman --no-pager` |
| `bundle audit` | CVE scanning | `bundle audit check --update` |

### Related
- Ruby Idioms @.gemini/skills/ruby-idioms/SKILL.md
- Database Design Principles @.gemini/skills/database-design-principles/SKILL.md
