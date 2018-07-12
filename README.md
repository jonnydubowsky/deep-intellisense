
An experimental implementation of IntelliSense for code using a trained recurrent neural network.

Should work on all kinds of code, but it needs at least ~50MB of code in the target language (more depending on syntax complexity) to be able to learn the syntax.

So far I've only tried it with TypesScript + React code. This is a pretty hard combination due to at least three different contexts with completely different rules (normal JS, TS Types, and JSX).

As is, this isn't good enough to be useful, but it shows that this kind of thing allows completions that would be hard to get otherwise. For example

```ts
import * as React ‸

// auto complete using learned prediction:

import * as React from "react";

// because almost all lines starting with `import * as React` end with `from "react";`

```

## Setup

1. Write random prettier config files for data augmentation

		node ./writeconfigs.js

    If you don't want data augmentation, just remove all prettierrc files except one.
2. Find matching open source repositories using GitHub API (output is included in repo)

		yarn install
		yarn run ts-node find.ts > repos.txt
3. Clone GitHub repos

		mkdir repos && cd repos
		../clone.sh ../repos.txt
4. Create input data file by merging all matching files from the cloned repos, formatted with each of the prettier configs
		
		./concat.sh
5. Clean the input data file (allcode.txt) to remove all non-ascii chars for a smaller output layer size 
		
		./clean.sh
6. Get char-rnn-tensorflow https://github.com/phiresky/char-rnn-tensorflow and train the net.

    My fork contains the "sample-stdin.py" script which allows sampling of strings via json IPC over stdin/stdout

    I trained it with this configuration:

    * 3 LSTM layers
    * 256 cells per layer (or try keeping it at 128)
    * 500 chars unrolled sequence length (or try 200)
    * batch size of 300
    * learning rate of 0.002 decaying at 0.99 per epoch


		python train.py --data_dir=./data/typescript-augmented --num_layers 3 --rnn_size 256 --num_epochs 100 --seq_length 500 --batch_size 300 --save_every 100 --decay_rate 0.99 --learning_rate 0.002 --save_dir save-aug-seq1000

    This will take a few hours on a GTX 980 Ti.

7. Open the [vscode-deep-complete](vscode-deep-complete/) directory in VS Code. Then run the extension from the debug panel.



