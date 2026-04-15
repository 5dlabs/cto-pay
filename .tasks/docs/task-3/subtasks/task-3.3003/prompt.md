<identity>
You are rex working on subtask 3003 of task 3.
</identity>

<context>
<scope>
Build the two instructions for AgentPackage lifecycle: register creates a new AgentPackage PDA with author validation and content_hash, and update allows the author to modify source_uri, content_hash, split_bps, and active status.
</scope>
</context>

<implementation_plan>
1. Create `programs/cto-billing/src/instructions/register_agent_package.rs`:
   - Define `RegisterAgentPackage` accounts struct: `author` (Signer, mut), `agent_package` (init, payer=author, space=AgentPackage::SIZE, seeds=[b"agent_package", package_id.as_bytes()], bump), `system_program`.
   - Instruction args: `package_id: String`, `split_bps: u16`, `source_uri: String`, `content_hash: [u8; 32]`.
   - Validation: `require!(package_id.len() <= 64, PackageIdTooLong)`, `require!(source_uri.len() <= 256, SourceUriTooLong)`, `require!(split_bps <= 10000, InvalidSplitBps)`, `require!(content_hash != [0u8; 32], InvalidContentHash)`.
   - Set all fields: author = author.key(), registered_at = Clock::get()?.unix_timestamp, active = true, total_earned = 0, task_count = 0, success_count = 0.
2. Create `programs/cto-billing/src/instructions/update_agent_package.rs`:
   - Define `UpdateAgentPackage` accounts struct: `author` (Signer), `agent_package` (mut, seeds=[b"agent_package", agent_package.package_id.as_bytes()], bump, has_one=author).
   - Instruction args: `source_uri: Option<String>`, `content_hash: Option<[u8; 32]>`, `split_bps: Option<u16>`, `active: Option<bool>`.
   - Apply only provided fields. Same validations for source_uri length and split_bps range. When source_uri is updated, require content_hash is also provided (and vice versa) to keep them in sync.
3. Register both instructions in `lib.rs` with appropriate `#[instruction]` attributes.
4. Export from `instructions/mod.rs`.
5. Test: Write a basic Anchor integration test in `tests/` that registers a package with split_bps=3000 and a valid content_hash, retrieves the PDA, and verifies all fields including content_hash. Test that a non-author cannot call update. Test that split_bps=10001 is rejected. Test that all-zero content_hash is rejected. Test that updating source_uri without content_hash fails.
</implementation_plan>

<validation>
Run `anchor test` — register_agent_package creates a PDA with correct seeds, all fields match input (package_id, split_bps=3000, source_uri, content_hash, author pubkey, active=true, counters=0, registered_at is recent timestamp). update_agent_package with new source_uri and content_hash succeeds when called by author, fails with ConstraintHasOne when called by different signer. register with split_bps=10001 fails with InvalidSplitBps. register with 65-char package_id fails with PackageIdTooLong. register with all-zero content_hash fails with InvalidContentHash. update with source_uri but no content_hash fails.
</validation>