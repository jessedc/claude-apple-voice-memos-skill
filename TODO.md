# TODO

## Features
- [ ] Convert to a Claude plugin as per guidance here: https://code.claude.com/docs/en/plugins#convert-existing-configurations-to-plugins

## Ideas
- [ ] Search within transcript content (currently only title search is supported)
- [ ] Title search via the metadata script (`--search` flag) instead of post-filtering in SKILL.md
- [ ] Batch transcript export to a single file (support the `all` flag end-to-end in scripts)

## Documentation
- [ ] Update SKILL with higher value prompt examples related to summarization 

## Quality
- [ ] Add basic tests for transcript extraction (known-good .m4a fixture)
- [ ] Add tests for metadata date filtering logic
- [ ] Add tests for disfluency cleaning and line-breaking algorithm
- [ ] Handle corrupt or truncated .m4a files gracefully in transcript extraction

## Housekeeping
- [ ] Add version number / release tagging scheme
