OBSEI x PIZZA 4P'S — DATASET DEMO

FILES
1) obsei_pizza_original_800.csv
   - 800 original synthetic chatbot conversations adapted from the BERTopic dataset.
   - The Obsei input column is `text`, populated from `noi_dung_khach_hang`.
   - Use this version for a fair comparison with the previous BERTopic experiment.

2) obsei_pizza_augmented_980.csv
   - The same 800 original conversations plus 180 new synthetic customer feedback statements.
   - The added records are balanced: 60 positive, 60 negative, 60 neutral.
   - Recommended for a broader Obsei demo covering intent, sentiment and themes.
   - Original 800 rows intentionally have blank sentiment_reference and intent_reference fields;
     their existing `reference_theme` is a topic/scenario reference, not a verified sentiment label.

3) obsei_eval_reference_180.csv
   - Labelled reference subset for optional classifier evaluation.
   - Contains only the 180 newly authored feedback records and their synthetic sentiment/intent labels.
   - These labels are synthetic reference labels, not real customer ground truth.

IMPORTANT
- All records are synthetic. They are not real Pizza 4P's customer data or verified reviews.
- The `text` column is the only text column intended for Obsei ingestion.
- The source CSV contains metadata columns for analysis; Obsei's CSV source may need field mapping
  if you want to preserve extra metadata. For the first `obsei try` preview, Obsei can read `text`.
- Keep the original BERTopic CSV unchanged as the baseline.
- Suggested first check:
    obsei try .\obsei_pizza_augmented_980.csv --limit 10
- Obsei 1.x Quickstart documents CSV text columns named text, comment, review, body, content,
  message or feedback.
