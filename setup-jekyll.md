# Quick Setup Guide for Running Jekyll Locally

## Step 1: Install Ruby
1. Download Ruby+Devkit from: https://rubyinstaller.org/downloads/
2. Install it (check "Add Ruby executables to your PATH")
3. When installation completes, run the MSYS2 development toolchain installer

## Step 2: Install Jekyll and Bundler
Open a new terminal and run:
```bash
gem install jekyll bundler
```

## Step 3: Install Project Dependencies
Navigate to your project directory and run:
```bash
bundle install
```

## Step 4: Run the Local Server
```bash
bundle exec jekyll serve
```

Then open http://localhost:4000 in your browser!

## Troubleshooting
If you get a `webrick` error, run:
```bash
bundle add webrick
```


