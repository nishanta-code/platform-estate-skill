# Brain project: columns, constraints and indexes for schema `brain`

Live snapshot taken 2026-11-24 from <DATA_REF>. Metadata only.
The brain schema holds the concierge/agent knowledge tables (9 tables, read-only to authenticated by design).
Legend: one line per table listing `column type` where a trailing `?` means NULLABLE (no `?` = NOT NULL), `=expr` is the column default (generated columns show their expression), and `/*...*/` is the column comment. Below each column line: `PK`/`FK`/`UQ`/`CK`/`EX` constraints (pg_get_constraintdef) and `idx` = indexes not backing a constraint (shown as `name [UNIQUE] USING method (cols) [WHERE ...]`).

## brain

### knowledge_assets
id uuid=gen_random_uuid(), slug text, type text, title text, description text?, file_url text?, link_url text?, program_id uuid?, is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK knowledge_assets_type_check: CHECK ((type = ANY (ARRAY['brochure'::text, 'flyer'::text, 'proposal'::text, 'one_pager'::text, 'application_link'::text, 'calendly'::text, 'sample_outcome'::text, 'other'::text])))
- FK knowledge_assets_program_id_fkey: FOREIGN KEY (program_id) REFERENCES brain.knowledge_programs(id) ON DELETE SET NULL
- PK knowledge_assets_pkey: PRIMARY KEY (id)
- UQ knowledge_assets_slug_key: UNIQUE (slug)
- idx knowledge_assets_program_idx USING btree (program_id)

### knowledge_base
key text, value text, description text?, updated_at timestamp with time zone=now()
- PK knowledge_base_pkey: PRIMARY KEY (key)

### knowledge_cadence
id uuid=gen_random_uuid(), n integer, offset_days integer, purpose text, target_words text?, rules text[]?='{}'::text[], is_active boolean=true, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK knowledge_cadence_pkey: PRIMARY KEY (id)
- UQ knowledge_cadence_n_key: UNIQUE (n)

### knowledge_faq
id uuid=gen_random_uuid(), question text, answer text, program_id uuid?, tags text[]?='{}'::text[], is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- FK knowledge_faq_program_id_fkey: FOREIGN KEY (program_id) REFERENCES brain.knowledge_programs(id) ON DELETE SET NULL
- PK knowledge_faq_pkey: PRIMARY KEY (id)
- idx knowledge_faq_program_idx USING btree (program_id)

### knowledge_intents
id uuid=gen_random_uuid(), tier text, code text, signals text[]?='{}'::text[], example text?, default_action text?, response_sla_hours integer?, is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK knowledge_intents_tier_check: CHECK ((tier = ANY (ARRAY['hot'::text, 'warm'::text, 'cold'::text])))
- PK knowledge_intents_pkey: PRIMARY KEY (id)
- UQ knowledge_intents_code_key: UNIQUE (code)

### knowledge_org
id uuid=gen_random_uuid(), name text, short_name text?, mission text?, parents text[]?='{}'::text[], website text?, agent_name text?, agent_email text?, agent_signature text?, cc_on_meeting_confirm text?, finlit_website text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK knowledge_org_pkey: PRIMARY KEY (id)

### knowledge_programs
id uuid=gen_random_uuid(), name text, target text?, format text?, mentors text?, pathways text[]?='{}'::text[], outcomes text?, differentiator text?, website text?, brochure_url text?, application_url text?, contact_email text?, is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), platform text?
- PK knowledge_programs_pkey: PRIMARY KEY (id)
- UQ knowledge_programs_name_key: UNIQUE (name)

### knowledge_scenarios
id uuid=gen_random_uuid(), slug text, name text, when_to_use text?, rules text[]?='{}'::text[], target_words text?, header_variant text?='info'::text, template_html text?, subject_hint text?, is_active boolean=true, order_index integer=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK knowledge_scenarios_pkey: PRIMARY KEY (id)
- UQ knowledge_scenarios_slug_key: UNIQUE (slug)

### knowledge_voice_rules
id uuid=gen_random_uuid(), kind text, text text, order_index integer=0, is_active boolean=true, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK knowledge_voice_rules_kind_check: CHECK ((kind = ANY (ARRAY['always'::text, 'never'::text, 'positioning'::text, 'hard_rule'::text])))
- PK knowledge_voice_rules_pkey: PRIMARY KEY (id)

