# NMT-based Corrective Patch Generation Dataset

Dataset for the study

Learning to Generate Corrective Patches using Neural Machine Translation

## Used tools for NMT

- [lamtram](https://github.com/neubig/lamtram)
- [dynet](https://github.com/clab/dynet)
- [fast_align](https://github.com/clab/fast_align)

Preparing a dictionary
```
  paste train/src.unk train/trg.unk | sed 's/	/ ||| /g' >   lamtram-table.txt
  fast_align/build/fast_align -i lamtram-table.txt -d -o -v -p lamtram- table.cond > lamtram-table.align
```

Training a model
```
  lamtram/src/lamtram/lamtram-train \
    --train_src train/src.unk \
    --train_trg train/trg.unk \
    --dynet-mem 2024 \
    --dropout 0.5 \
    --rate_decay 0.5 \
    --model_type encatt \
    --attention_type mlp:0 \
    --attention_feed true \
    --attention_hist none \
    --trainer adam \
    --learning_rate 0.001 \
    --layers lstm \
    --wordrep 512 \
    --encoder_types "for|rev" \
    --attention_lex "prior:file=lamtram-table.probunk:alpha=0.001" \
    --minibatch_size 2048 \
    --seed $SEED \
    --model_out lamtram-model.$SEED.mod
```

Generating statements
```
  lamtram/src/lamtram/lamtram \
    --dynet-mem 2024 \
    --operation gen \
    --models_in "encatt=lamtram-model.100.mod|encatt=lamtram-model.200.mod|encatt=lamtram-model.300.mod" \
    --src_in test/buggy-NU_src.txt \
    --map_in lamtram-table.prob \
    --nbest_size 10 \
    --beam 10
```
