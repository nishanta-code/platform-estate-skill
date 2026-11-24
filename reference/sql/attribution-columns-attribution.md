# Brain project: columns, constraints and indexes for schema `attribution`

Live snapshot taken 2026-11-24 from <DATA_REF>. Metadata only.
The attribution schema is the UTM/outreach attribution model (7 tables, 4 views, 0 RLS policies, CHECK-enforced id formats).
Legend: one line per table listing `column type` where a trailing `?` means NULLABLE (no `?` = NOT NULL), `=expr` is the column default (generated columns show their expression), and `/*...*/` is the column comment. Below each column line: `PK`/`FK`/`UQ`/`CK`/`EX` constraints (pg_get_constraintdef) and `idx` = indexes not backing a constraint (shown as `name [UNIQUE] USING method (cols) [WHERE ...]`).

## attribution

### campaigns
campaign_id text, name text, offer text?, utm_campaign text, price_usd numeric?, apply_url text?, details_url text?, papers_url text?, active boolean=false, created_at timestamp with time zone=now()
- CK campaigns_apply_url_check: CHECK ((apply_url ~ '^https://[^?#]+$'::text))
- CK campaigns_campaign_id_check: CHECK ((campaign_id ~ '^[A-Z]{2,4}-[A-Z0-9]+(-[A-Z0-9]+)*$'::text))
- CK campaigns_check: CHECK (((NOT active) OR (apply_url IS NOT NULL)))
- CK campaigns_details_url_check: CHECK ((details_url ~ '^https://[^?#]+$'::text))
- CK campaigns_papers_url_check: CHECK ((papers_url ~ '^https://[^?#]+$'::text))
- CK campaigns_utm_campaign_check: CHECK ((utm_campaign ~ '^[a-z0-9_]+$'::text))
- PK campaigns_pkey: PRIMARY KEY (campaign_id)
- UQ campaigns_utm_campaign_key: UNIQUE (utm_campaign)

### circulation_overrides
school_id text, campaign_id text, status text, set_by text, set_at timestamp with time zone=now(), note text?
- CK circulation_overrides_status_check: CHECK ((status = ANY (ARRAY['CIRCULATION CONFIRMED'::text, 'DECLINED'::text, 'NO RESPONSE'::text])))
- FK circulation_overrides_campaign_id_fkey: FOREIGN KEY (campaign_id) REFERENCES attribution.campaigns(campaign_id)
- FK circulation_overrides_school_id_fkey: FOREIGN KEY (school_id) REFERENCES attribution.schools(school_id)
- PK circulation_overrides_pkey: PRIMARY KEY (school_id, campaign_id)

### contacts
contact_id text, school_id text, full_name text, utm_name text, role text?, email text?, contact_status text, email_status text, outreach_use text?, best_offer text?, verification_source text?, verified_on date?, notes text?, active boolean=true, source_code text, created_at timestamp with time zone=now()
- CK contacts_check: CHECK (((email_status <> 'VERIFIED'::text) OR (email IS NOT NULL)))
- CK contacts_contact_id_check: CHECK ((contact_id ~ '^[A-Z]{3}-[A-Z0-9]{2,6}-C[0-9]{2}$'::text))
- CK contacts_utm_name_check: CHECK ((utm_name ~ '^[a-z0-9_]+$'::text))
- FK contacts_school_id_fkey: FOREIGN KEY (school_id) REFERENCES attribution.schools(school_id)
- FK contacts_source_code_fkey: FOREIGN KEY (source_code) REFERENCES source_codes(code)
- PK contacts_pkey: PRIMARY KEY (contact_id)
- UQ contacts_school_id_utm_name_key: UNIQUE (school_id, utm_name)
- idx contacts_email_uq UNIQUE USING btree (lower(email)) WHERE (email IS NOT NULL)

### outreach
outreach_id text, contact_id text, campaign_id text, touch_no integer, status text='DRAFT'::text, approved_to_send boolean=false, approved_by text?, approved_at timestamp with time zone?, parallel_approved_by text?, subject text?, sent_at timestamp with time zone?, gmail_message_id text?, gmail_thread_id text?, reply_status text='NO REPLY'::text, last_reply_at timestamp with time zone?, follow_up_due date?, suppressed boolean=false, suppression_reason text?, is_test boolean=false, created_by text, created_at timestamp with time zone=now()
- CK outreach_check: CHECK (((NOT approved_to_send) OR (approved_by IS NOT NULL)))
- CK outreach_reply_status_check: CHECK ((reply_status = ANY (ARRAY['NO REPLY'::text, 'REPLIED'::text, 'UNSUBSCRIBED'::text, 'BOUNCED'::text, 'HUMAN TAKEOVER'::text])))
- CK outreach_status_check: CHECK ((status = ANY (ARRAY['DRAFT'::text, 'APPROVED'::text, 'SENT'::text, 'CANCELLED'::text])))
- CK outreach_touch_no_check: CHECK (((touch_no >= 1) AND (touch_no <= 99)))
- FK outreach_campaign_id_fkey: FOREIGN KEY (campaign_id) REFERENCES attribution.campaigns(campaign_id)
- FK outreach_contact_id_fkey: FOREIGN KEY (contact_id) REFERENCES attribution.contacts(contact_id)
- PK outreach_pkey: PRIMARY KEY (outreach_id)
- UQ outreach_contact_id_touch_no_key: UNIQUE (contact_id, touch_no)

### schools
school_id text, name text, utm_source text, city text?, state text?, country text='USA'::text, cluster text?, created_at timestamp with time zone=now()
- CK schools_school_id_check: CHECK ((school_id ~ '^[A-Z]{3}-[A-Z0-9]{2,6}$'::text))
- CK schools_utm_source_check: CHECK ((utm_source ~ '^[a-z0-9_]+$'::text))
- PK schools_pkey: PRIMARY KEY (school_id)
- UQ schools_utm_source_key: UNIQUE (utm_source)

### settings
key text, value text, note text?
- PK settings_pkey: PRIMARY KEY (key)

### suppressions
email text, campaign_id text?, reason text, created_at timestamp with time zone=now()
- FK suppressions_campaign_id_fkey: FOREIGN KEY (campaign_id) REFERENCES attribution.campaigns(campaign_id)
- idx suppressions_uq UNIQUE USING btree (lower(email), COALESCE(campaign_id, '*'::text))

