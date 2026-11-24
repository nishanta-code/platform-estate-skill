# Brain project: columns, constraints and indexes for schema `quarantine`

Live snapshot taken 2026-11-24 from <DATA_REF>. Metadata only.
The quarantine schema holds scraped/unvalidated research data that is NOT yet people/organizations (14 tables, 0 RLS policies). Promotion into public.* is gated by CHECK constraints and functions such as email_sendability() and meeting_dataset_pairs().
Legend: one line per table listing `column type` where a trailing `?` means NULLABLE (no `?` = NOT NULL), `=expr` is the column default (generated columns show their expression), and `/*...*/` is the column comment. Below each column line: `PK`/`FK`/`UQ`/`CK`/`EX` constraints (pg_get_constraintdef) and `idx` = indexes not backing a constraint (shown as `name [UNIQUE] USING method (cols) [WHERE ...]`).

## quarantine

### dataset_rows
dataset_id uuid, research_contact_id uuid, added_at timestamp with time zone=now()
- FK dataset_rows_dataset_id_fkey: FOREIGN KEY (dataset_id) REFERENCES quarantine.datasets(id) ON DELETE CASCADE
- FK dataset_rows_research_contact_id_fkey: FOREIGN KEY (research_contact_id) REFERENCES quarantine.research_contacts(id) ON DELETE CASCADE
- PK dataset_rows_pkey: PRIMARY KEY (dataset_id, research_contact_id)
- idx dataset_rows_contact_ix USING btree (research_contact_id)

### datasets
id uuid=gen_random_uuid(), code text, location_country text, location_state text?, location_city text?, built_by uuid, built_at timestamp with time zone=now(), redo_of uuid?, status text='building'::text, campaign_done_at timestamp with time zone?, note text?
- CK dataset_campaign_done_complete: CHECK (((status = 'campaign_done'::text) = (campaign_done_at IS NOT NULL)))
- CK dataset_redo_not_self: CHECK (((redo_of IS NULL) OR (redo_of <> id)))
- CK datasets_code_check: CHECK ((code ~ '^[A-Z0-9-]{2,24}$'::text))
- CK datasets_status_check: CHECK ((status = ANY (ARRAY['building'::text, 'ready'::text, 'campaign_done'::text, 'superseded'::text])))
- FK datasets_built_by_fkey: FOREIGN KEY (built_by) REFERENCES staff(id)
- FK datasets_redo_of_fkey: FOREIGN KEY (redo_of) REFERENCES quarantine.datasets(id)
- PK datasets_pkey: PRIMARY KEY (id)
- UQ datasets_code_key: UNIQUE (code)
- idx datasets_location_ix USING btree (lower(btrim(location_country)), lower(btrim(COALESCE(location_state, ''::text))))

### email_domain_evidence
domain text, method text, result text, detail jsonb?, checked_at timestamp with time zone=now(), checked_by text
- CK email_domain_evidence_method_check: CHECK ((method = ANY (ARRAY['dns_resolve'::text, 'domain_mx'::text, 'freemail'::text, 'disposable'::text, 'catch_all'::text, 'parked'::text])))
- CK email_domain_evidence_result_check: CHECK ((result = ANY (ARRAY['pass'::text, 'fail'::text, 'unknown'::text])))
- PK email_domain_evidence_pkey: PRIMARY KEY (domain, method)
- idx email_domain_evidence_stale_ix USING btree (method, checked_at)

### email_evidence
id uuid=gen_random_uuid(), email_norm text, method text, result text, detail jsonb?, checked_at timestamp with time zone=now(), checked_by text
- CK email_evidence_method_check: CHECK ((method = ANY (ARRAY['syntax'::text, 'domain_mx'::text, 'role_account'::text, 'disposable'::text, 'catch_all'::text, 'smtp_probe'::text, 'provider_api'::text, 'engagement'::text, 'bounce_soft'::text, 'bounce_hard'::text, 'complaint'::text, 'trap_hit'::text, 'plausibility'::text, 'freemail'::text, 'ai_review'::text])))
- CK email_evidence_norm_not_blank: CHECK ((btrim(email_norm) <> ''::text))
- CK email_evidence_result_check: CHECK ((result = ANY (ARRAY['pass'::text, 'fail'::text, 'unknown'::text])))
- PK email_evidence_pkey: PRIMARY KEY (id)
- idx email_evidence_addr_ix USING btree (email_norm, checked_at DESC)
- idx email_evidence_method_ix USING btree (method, checked_at DESC)
- idx email_evidence_fatal_ix USING btree (email_norm) WHERE (method = ANY (ARRAY['bounce_hard'::text, 'complaint'::text, 'trap_hit'::text]))
- idx email_evidence_addr_method_uk UNIQUE USING btree (email_norm, method)

### geo_aliases
raw_state text, kind text, rows_affected integer, countries jsonb='[]'::jsonb, sample_cities jsonb='[]'::jsonb, canonical_state text?, canonical_country text?, confidence numeric(4,3)?, reason text, proposed_by text, proposed_at timestamp with time zone=now(), verdict text?, decided_by uuid?, decided_at timestamp with time zone?, applied_at timestamp with time zone?
- CK geo_aliases_kind_check: CHECK ((kind = ANY (ARRAY['country_in_state_field'::text, 'country_name'::text, 'two_letter_code'::text, 'numeric'::text, 'over_long'::text, 'plausible'::text, 'unknown'::text])))
- CK geo_aliases_verdict_check: CHECK (((verdict IS NULL) OR (verdict = ANY (ARRAY['approved'::text, 'rejected'::text, 'held'::text]))))
- FK geo_aliases_decided_by_fkey: FOREIGN KEY (decided_by) REFERENCES staff(id)
- PK geo_aliases_pkey: PRIMARY KEY (raw_state)
- idx geo_aliases_queue_ix USING btree (rows_affected DESC) WHERE (verdict IS NULL)
- idx geo_aliases_apply_ix USING btree (verdict) WHERE ((verdict = 'approved'::text) AND (applied_at IS NULL))

### import_batches
id uuid=gen_random_uuid(), label text, file_name text?, header_fingerprint text, headers text[], column_map jsonb='{}'::jsonb, map_source text='guessed'::text, uploaded_by uuid, uploaded_at timestamp with time zone=now(), committed_at timestamp with time zone?, committed_by uuid?, stopped_reason text?, kind text='research_contact'::text /*What the rows become. Guessed from the headers by import_guess_kind() and changeable by hand — the first real file was a sheet of schools put through a contacts mapper.*/
- CK import_batch_commit_complete: CHECK (((committed_at IS NULL) = (committed_by IS NULL)))
- CK import_batches_kind_check: CHECK ((kind = ANY (ARRAY['research_contact'::text, 'organisation'::text])))
- CK import_batches_map_source_check: CHECK ((map_source = ANY (ARRAY['guessed'::text, 'remembered'::text, 'set_by_hand'::text])))
- FK import_batches_committed_by_fkey: FOREIGN KEY (committed_by) REFERENCES staff(id)
- FK import_batches_uploaded_by_fkey: FOREIGN KEY (uploaded_by) REFERENCES staff(id)
- PK import_batches_pkey: PRIMARY KEY (id)

### import_header_maps
header_fingerprint text, headers text[], column_map jsonb, times_used integer=1, updated_by uuid?, updated_at timestamp with time zone=now()
- FK import_header_maps_updated_by_fkey: FOREIGN KEY (updated_by) REFERENCES staff(id)
- PK import_header_maps_pkey: PRIMARY KEY (header_fingerprint)

### import_rows
id uuid=gen_random_uuid(), batch_id uuid, row_no integer, raw jsonb, verdict text?, reason text?, research_contact_id uuid?, decided_at timestamp with time zone?, org_candidate_id uuid?
- CK import_row_decision_complete: CHECK (((verdict IS NULL) = (decided_at IS NULL)))
- CK import_rows_verdict_check: CHECK ((verdict = ANY (ARRAY['saved'::text, 'refused'::text, 'duplicate'::text, 'incomplete'::text, 'malformed'::text])))
- FK import_rows_batch_id_fkey: FOREIGN KEY (batch_id) REFERENCES quarantine.import_batches(id) ON DELETE CASCADE
- FK import_rows_org_candidate_id_fkey: FOREIGN KEY (org_candidate_id) REFERENCES quarantine.org_candidates(id) ON DELETE SET NULL
- FK import_rows_research_contact_id_fkey: FOREIGN KEY (research_contact_id) REFERENCES quarantine.research_contacts(id) ON DELETE SET NULL
- PK import_rows_pkey: PRIMARY KEY (id)
- UQ import_rows_batch_id_row_no_key: UNIQUE (batch_id, row_no)
- idx import_rows_batch_ix USING btree (batch_id, row_no)

### org_candidates
id uuid=gen_random_uuid(), domain text?, company_name text, people_count integer, name_agreement boolean /*False when the people on this domain name different employers — the umbrella signal, and the reason the promotion CHECK exists. It is what separates chula.ac.th (611 people, one university, promotable) from dallasisd.org (76 people naming both the district and Bryan Adams High School inside it, refused). Counter-intuitively the LARGEST clusters conflict most: 29% of 50+ clusters carry several names against 10% of 2-person ones, because a big shared domain is usually a district or a university rather than one school. But a 2-person unanimous cluster is weak evidence too — two rows from the same purchased list with the same company field are not two witnesses. The 5-49 band is the sweet spot: 285 unanimous clusters, large enough to corroborate and small enough to be one body.*/, website text? /*The website the people on this domain state, from Apollo enrichment. Its value is as a SECOND, independent corroboration: website = domain holds for 1,364 of 1,646 candidates (83%), the same rate measured across all 10,346 prospects before any of this ran. Where the two differ, look closer — that is also how a misspelled free-mail domain betrays itself, since the stated site will be the real employer's rather than the typo.*/, industry text?, linkedin_url text?, existing_organization_id uuid? /*The domain is already reachable from this organisation, either because it matches organizations.domain or through an exclusive affiliated mailbox. LINK to it; never create. 21 of the first 1,646 candidates are in this state (18 by domain, 3 by mailbox). That is a FLOOR, not an estimate — it is only measurable against the 98 organisations that have an affiliated person. The other 252 have nobody attached, so a collision with them cannot be detected at all. Supersedes the figure of 120 in 016, which came from a query whose join fanned out.*/, promoted_organization_id uuid?, promoted_at timestamp with time zone?, landed_at timestamp with time zone=now(), country_raw text?, city_raw text?, requested_by uuid? /*The staff member who could not find a school and asked for it. NULL for candidates the transforms discovered. 036.*/, request_note text?, reviewed_by uuid?, reviewed_at timestamp with time zone?, review_note text?, source_system text?, source_id text?, kind text? /*What the domain is, decided by how many distinct school names sit on it. A district is not a school and must never be promoted as one.*/, confidence numeric(4,3)? /*On org_aliases existing scale: 1.000 exact key match, 0.850 single name on a live domain, 0.750 several names. NULL for a district or a consumer mailbox, where the question does not apply.*/, name_variants integer?, classified_at timestamp with time zone?, classified_by text?
- CK org_candidates_kind_check: CHECK (((kind IS NULL) OR (kind = ANY (ARRAY['school'::text, 'school_variants'::text, 'district'::text, 'consumer'::text, 'unresolvable'::text]))))
- CK org_candidates_people_positive: CHECK ((people_count > 0))
- CK org_candidates_promotion_complete: CHECK (((promoted_organization_id IS NULL) = (promoted_at IS NULL)))
- CK org_candidates_promotion_earned: CHECK (((promoted_organization_id IS NULL) OR (name_agreement AND (existing_organization_id IS NULL))))
- FK org_candidates_existing_organization_id_fkey: FOREIGN KEY (existing_organization_id) REFERENCES organizations(id) ON DELETE SET NULL
- FK org_candidates_promoted_organization_id_fkey: FOREIGN KEY (promoted_organization_id) REFERENCES organizations(id) ON DELETE SET NULL
- FK org_candidates_requested_by_fkey: FOREIGN KEY (requested_by) REFERENCES staff(id)
- FK org_candidates_reviewed_by_fkey: FOREIGN KEY (reviewed_by) REFERENCES staff(id)
- PK org_candidates_pkey: PRIMARY KEY (id)
- UQ org_candidates_domain_uk: UNIQUE (domain)
- idx org_candidates_promotable_idx USING btree (people_count DESC) WHERE (name_agreement AND (existing_organization_id IS NULL) AND (promoted_organization_id IS NULL))
- idx org_candidates_link_idx USING btree (existing_organization_id) WHERE (existing_organization_id IS NOT NULL)
- idx org_candidates_conflict_idx USING btree (people_count DESC) WHERE (NOT name_agreement)
- idx org_candidates_promoted_idx USING btree (promoted_organization_id) WHERE (promoted_organization_id IS NOT NULL)
- idx org_candidates_name_country_ix USING btree (lower(btrim(company_name)), lower(btrim(COALESCE(country_raw, ''::text)))) WHERE (promoted_organization_id IS NULL)
- idx org_candidates_source_ux UNIQUE USING btree (source_system, source_id) WHERE ((source_system IS NOT NULL) AND (source_id IS NOT NULL))

### rejected_identities
id uuid=gen_random_uuid(), person_id uuid, kind text, value_raw text, reason text, source_code text?, moved_at timestamp with time zone=now()
- FK rejected_identities_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id)
- PK rejected_identities_pkey: PRIMARY KEY (id)

### research_contact_edits
id uuid=gen_random_uuid(), contact_id uuid, field text, old_value text?, new_value text?, reason text, edited_by uuid?, edited_at timestamp with time zone=now()
- FK research_contact_edits_contact_id_fkey: FOREIGN KEY (contact_id) REFERENCES quarantine.research_contacts(id) ON DELETE CASCADE
- FK research_contact_edits_edited_by_fkey: FOREIGN KEY (edited_by) REFERENCES staff(id) ON DELETE SET NULL
- PK research_contact_edits_pkey: PRIMARY KEY (id)
- idx research_contact_edits_contact_ix USING btree (contact_id, edited_at DESC)
- idx research_contact_edits_field_ix USING btree (field, edited_at DESC)

### research_contacts
id uuid=gen_random_uuid(), source_system text='legacy'::text, source_table text='old_research_data'::text, source_id text, email_norm text?, email_raw text?, full_name text?, phone_raw text?, school_name_raw text?, country_raw text?, city_raw text?, state_raw text?, continent_raw text?, is_valid boolean? /*Three-valued on purpose. NULL = never looked at. false = checked and rejected. true = passed. Only true may be promoted.*/, status text?, validation_type text? /*How it was checked. Every validated row in the source says 'mx' — an MX record check proves the DOMAIN accepts mail, NOT that the mailbox exists. It does not prevent a bounce.*/, verified_at timestamp with time zone?, errors jsonb?, retry_count integer=0, promoted_person_id uuid?, promoted_at timestamp with time zone?, organization_id uuid?, landed_at timestamp with time zone=now(), designation_raw text? /*The role as the school states it. Named in the required format on the call - and it is exactly the column that used to receive a phone number, so the desk enforces its position rather than cleaning it later.*/, added_by uuid? /*The staff member whose keystrokes or upload produced this row. NULL on the 647,650 that predate the desk — that is the truth, not a backfill opportunity.*/, review_verdict text?, review_reason text?, reviewed_by uuid?, reviewed_at timestamp with time zone?, state_norm text? /*The reviewed state, beside the raw value rather than over it. NULL means either not yet decided or decided to be unknowable — geo_aliases carries which.*/, country_norm text?, geo_norm_at timestamp with time zone?
- CK quarantine_promotion_complete: CHECK (((promoted_person_id IS NULL) = (promoted_at IS NULL)))
- CK quarantine_promotion_requires_validation: CHECK (((promoted_person_id IS NULL) OR ((is_valid IS TRUE) AND (verified_at IS NOT NULL))))
- CK research_contact_review_complete: CHECK (((review_verdict IS NULL) = (reviewed_at IS NULL)))
- CK research_contacts_review_verdict_check: CHECK ((review_verdict = ANY (ARRAY['approved'::text, 'rejected'::text, 'held'::text])))
- FK research_contacts_added_by_fkey: FOREIGN KEY (added_by) REFERENCES staff(id) ON DELETE SET NULL
- FK research_contacts_organization_id_fkey: FOREIGN KEY (organization_id) REFERENCES organizations(id) ON DELETE SET NULL
- FK research_contacts_promoted_person_id_fkey: FOREIGN KEY (promoted_person_id) REFERENCES people(id) ON DELETE SET NULL
- FK research_contacts_reviewed_by_fkey: FOREIGN KEY (reviewed_by) REFERENCES staff(id) ON DELETE SET NULL
- PK research_contacts_pkey: PRIMARY KEY (id)
- UQ research_contacts_source_uk: UNIQUE (source_system, source_table, source_id)
- idx research_contacts_pending_idx USING btree (landed_at) WHERE ((is_valid IS NULL) AND (promoted_person_id IS NULL))
- idx research_contacts_promotable_idx USING btree (verified_at) WHERE ((is_valid IS TRUE) AND (promoted_person_id IS NULL))
- idx research_contacts_email_idx USING btree (email_norm) WHERE (email_norm IS NOT NULL)
- idx research_contacts_promoted_idx USING btree (promoted_person_id) WHERE (promoted_person_id IS NOT NULL)
- idx research_contacts_school_trgm USING gin (school_name_raw gin_trgm_ops)
- idx research_contacts_org_idx USING btree (organization_id) WHERE (organization_id IS NOT NULL)
- idx research_contacts_school_trgm_ix USING gin (lower(btrim(school_name_raw)) gin_trgm_ops)
- idx research_contacts_country_ix USING btree (lower(btrim(country_raw)))
- idx research_contacts_state_ix USING btree (is_valid, promoted_person_id, organization_id)
- idx research_contacts_added_by_ix USING btree (added_by, landed_at DESC) WHERE (added_by IS NOT NULL)
- idx research_contacts_review_ix USING btree (review_verdict, reviewed_at DESC) WHERE (review_verdict IS NOT NULL)
- idx research_contacts_state_norm_ix USING btree (lower(state_norm)) WHERE (state_norm IS NOT NULL)
- idx research_contacts_domain_link_ix USING btree (lower(split_part(email_norm, '@'::text, 2))) WHERE ((organization_id IS NULL) AND (promoted_person_id IS NULL))
- idx research_contacts_unlinked_landed_ix USING btree (landed_at DESC) WHERE ((organization_id IS NULL) AND (promoted_person_id IS NULL))

### research_schools
name_key text, name text, country_raw text? /*The most commonly claimed country among this school's contacts. NOT a fact — for two of the largest schools in this data the modal country is demonstrably wrong. Read alongside country_variants and country_agreement, and confirm with a human before promoting. 036.*/, city_raw text?, contacts integer=0, valid_contacts integer=0, organization_id uuid?, candidate_id uuid?, first_seen timestamp with time zone?, refreshed_at timestamp with time zone=now(), is_provider boolean=false /*True when the name is an email provider or a spreadsheet placeholder rather than a school. Marked, not deleted. Excluded from the review queue by default, visible on request. 036.*/, triage_reason text?, country_agreement numeric(4,3)?, country_variants integer=0 /*How many distinct countries the contacts claim for this school. >1 means the location is disputed and must not be presented as settled. 036.*/, rejected_at timestamp with time zone? /*A human decided this name is not a school we will hold. Never deleted — the research rows are untouched and the reason is kept, so the decision is auditable and does not have to be made twice. 036.*/, rejected_by uuid?, reject_reason text? /*Why a person said this name is not a school. Required — reject_research_school() refuses a blank one, because "not a school" in three months tells nobody why. 098.*/
- FK research_schools_organization_id_fkey: FOREIGN KEY (organization_id) REFERENCES organizations(id)
- FK research_schools_rejected_by_fkey: FOREIGN KEY (rejected_by) REFERENCES staff(id)
- PK research_schools_pkey: PRIMARY KEY (name_key)
- idx research_schools_trgm_ix USING gin (name_key gin_trgm_ops)
- idx research_schools_country_ix USING btree (lower(btrim(COALESCE(country_raw, ''::text))))
- idx research_schools_unpromoted_ix USING btree (contacts DESC) WHERE (organization_id IS NULL)
- idx research_schools_reviewable_ix USING btree (contacts DESC) WHERE ((organization_id IS NULL) AND (NOT is_provider))
- idx research_schools_open_ix USING btree (contacts DESC) WHERE ((organization_id IS NULL) AND (rejected_at IS NULL) AND (NOT is_provider))

### validator_runs
id uuid=gen_random_uuid(), agent text, stage text, started_at timestamp with time zone=now(), finished_at timestamp with time zone?, domains_checked integer=0, addresses_hit integer=0, passed integer=0, failed integer=0, errors integer=0, detail jsonb?
- PK validator_runs_pkey: PRIMARY KEY (id)
- idx validator_runs_recent_ix USING btree (agent, started_at DESC)

