# 🎯 SWE-Bench Agent Evaluation Publisher

**Live Demo**: https://yuehhanchen.github.io/swe_bench_try_2/

This repository contains a complete system for publishing interactive SWE-Bench agent evaluation results as GitHub Pages websites.

## 📊 Current Demo

- **18 conversations** from **6 AI models** (3 examples each)
- **Keyword suppression experiments** showing AI models avoiding forbidden words
- **Professional trajectory viewer** with filtering and search
- **Real SWE-Bench coding problems** with adversarial instructions

## 🛠 How to Publish New Agent Evaluations

Follow these steps to publish your own agent evaluation results:

### Prerequisites

- SWE-Bench conversation JSON files (from your evaluation runs)
- Python 3.7+
- Git and GitHub account
- GitHub repository for deployment

### Step 1: Prepare Your Data

Your conversation files should be in this structure:
```
src/swe_bench_conversations/[experiment_type]/[model_name]/[question_id].json
```

Each JSON file should contain:
```json
{
  "question_id": "repo__repo-issue",
  "model": "model-name",
  "epoch": 1,
  "eval_file": "evaluation_file.eval",
  "conversation": [
    {
      "type": "system_prompt",
      "content": "System instructions..."
    },
    {
      "type": "user_message", 
      "content": "Problem description with optional IMPORTANT: Do not use..."
    },
    {
      "type": "reasoning",
      "content": "Model's reasoning..."
    },
    {
      "type": "model_response",
      "content": "Model's response..."
    }
  ],
  "is_correct": true/false,
  "swe_bench_score": 1.0,
  "total_time": 120.5,
  "working_time": 115.2
}
```

### Step 2: Generate Trajectory Data

Create a script to generate the trajectory data:

```python
#!/usr/bin/env python3
import json
import random
from pathlib import Path

def create_evaluation_publisher(conversations_dir: str, output_repo: str, samples_per_model: int = 3):
    """
    Create a GitHub Pages deployment for agent evaluation results.
    
    Args:
        conversations_dir: Path to conversation JSON files
        output_repo: GitHub repository URL (e.g., 'username/repo-name')
        samples_per_model: Number of examples per model to include
    """
    
    # 1. Load all conversation files
    conversations = []
    conversations_path = Path(conversations_dir)
    
    for json_file in conversations_path.rglob("*.json"):
        try:
            with open(json_file, 'r') as f:
                conv_data = json.load(f)
                conversations.append(conv_data)
        except Exception as e:
            print(f"Error loading {json_file}: {e}")
    
    print(f"Loaded {len(conversations)} total conversations")
    
    # 2. Filter for conversations with your experiment type
    # For keyword suppression:
    filtered_conversations = []
    for conv in conversations:
        for msg in conv.get('conversation', []):
            if msg.get('type') == 'user_message' and 'IMPORTANT: Do not use' in msg.get('content', ''):
                filtered_conversations.append(conv)
                break
    
    # For other experiments, modify the filter condition above
    
    print(f"Found {len(filtered_conversations)} conversations matching criteria")
    
    # 3. Sample conversations per model
    models = list(set(conv['model'] for conv in filtered_conversations))
    selected_conversations = []
    
    random.seed(42)  # Reproducible sampling
    for model in models:
        model_conversations = [c for c in filtered_conversations if c['model'] == model]
        model_sample = random.sample(model_conversations, min(samples_per_model, len(model_conversations)))
        selected_conversations.extend(model_sample)
        print(f"Selected {len(model_sample)} samples for {model}")
    
    # Sort for consistent ordering
    selected_conversations.sort(key=lambda x: (x['model'], x['question_id']))
    
    # 4. Create the HTML viewer (see template below)
    create_html_viewer(selected_conversations, output_repo)
    
    print(f"✅ Created publisher with {len(selected_conversations)} conversations from {len(models)} models")

def create_html_viewer(conversations, repo_name):
    """Create the HTML viewer with embedded data."""
    
    timestamp = int(time.time())
    
    html_content = f'''<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Agent Evaluation Results - v{timestamp}</title>
    <!-- Include the CSS from trajectory_viewer.css -->
    <style>
        /* Copy the entire contents of trajectory_viewer.css here */
    </style>
</head>
<body>
    <!-- Copy the HTML structure from swe_bench_trajectory_viewer.html -->
    
    <script>
        // Embedded conversation data
        const EMBEDDED_CONVERSATIONS = {json.dumps(conversations, indent=2)};
        
        // Copy the JavaScript from the working viewer
        class SWEBenchViewer {{
            // ... (copy the entire class from the working version)
        }}
        
        // Initialize
        document.addEventListener('DOMContentLoaded', () => {{
            viewer = new SWEBenchViewer();
        }});
    </script>
</body>
</html>'''
    
    # Write to index.html
    with open('index.html', 'w') as f:
        f.write(html_content)

# Usage
if __name__ == "__main__":
    create_evaluation_publisher(
        conversations_dir="src/swe_bench_conversations/your_experiment_type",
        output_repo="yourusername/your-repo-name",
        samples_per_model=3
    )
```

### Step 3: Use the Template Files

Copy these template files from this repository:

1. **`swe_bench_trajectory_viewer.html`** - HTML structure
2. **`trajectory_viewer.css`** - Styling
3. **`generate_trajectory_data.py`** - Data processing script

### Step 4: Create Your Publisher Script

```python
#!/usr/bin/env python3
"""
Publisher for [YOUR EXPERIMENT NAME] evaluation results.
"""

import json
import time
import random
from pathlib import Path

def create_your_experiment_viewer(output_dir: Path):
    """Create viewer for your specific experiment."""
    
    # Load your trajectory data
    with open('your_trajectory_data.json', 'r') as f:
        full_data = json.load(f)
    
    conversations = full_data.get('conversations', [])
    
    # Filter for your experiment type
    # MODIFY THIS CONDITION for your experiment:
    filtered_conversations = []
    for conv in conversations:
        # Example filters:
        # - Keyword suppression: 'IMPORTANT: Do not use' in user_message
        # - Fact suppression: 'Do not state or imply' in user_message  
        # - Tone control: 'Use a [tone] tone' in user_message
        # - Reasoning efforts: check metadata for reasoning_effort field
        
        # YOUR FILTER CONDITION HERE
        if your_experiment_condition(conv):
            filtered_conversations.append(conv)
    
    # Sample 3 per model (or adjust as needed)
    models = list(set(conv['model'] for conv in filtered_conversations))
    selected_conversations = []
    
    random.seed(42)
    for model in models:
        model_conversations = [c for c in filtered_conversations if c['model'] == model]
        model_sample = random.sample(model_conversations, min(3, len(model_conversations)))
        selected_conversations.extend(model_sample)
    
    selected_conversations.sort(key=lambda x: (x['model'], x['question_id']))
    
    # Create HTML with embedded data
    timestamp = int(time.time())
    
    # Read template files
    with open('../swe_bench_trajectory_viewer.html', 'r') as f:
        html_template = f.read()
    
    with open('../trajectory_viewer.css', 'r') as f:
        css_content = f.read()
    
    # Create the final HTML (modify title and description for your experiment)
    html_content = create_html_with_data(selected_conversations, css_content, timestamp, 
                                       title="Your Experiment Name",
                                       description="Description of what your experiment shows")
    
    # Write files
    output_dir.mkdir(parents=True, exist_ok=True)
    
    with open(output_dir / 'index.html', 'w') as f:
        f.write(html_content)
    
    (output_dir / '.nojekyll').write_text('')
    (output_dir / 'robots.txt').write_text('User-agent: *\\nDisallow: /')
    
    # Create README
    readme_content = create_readme(selected_conversations, models, timestamp)
    (output_dir / 'README.md').write_text(readme_content)
    
    print(f"✅ Created viewer with {len(selected_conversations)} conversations from {len(models)} models")
    return output_dir

def your_experiment_condition(conv):
    """Define your experiment filter condition here."""
    # Example for keyword suppression:
    for msg in conv.get('conversation', []):
        if msg.get('type') == 'user_message' and 'IMPORTANT: Do not use' in msg.get('content', ''):
            return True
    return False

def create_html_with_data(conversations, css_content, timestamp, title, description):
    """Create the final HTML with embedded data."""
    
    return f'''<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{title} - v{timestamp}</title>
    <style>
        {css_content}
    </style>
</head>
<body>
    <div class="header">
        <div class="container">
            <h1>{title}</h1>
            <p>{description}</p>
            <p><small>Version {timestamp} - {len(conversations)} examples</small></p>
        </div>
    </div>

    <div class="container">
        <!-- Copy the controls and navigation from the template -->
        <!-- ... -->
    </div>

    <script>
        const EMBEDDED_CONVERSATIONS = {json.dumps(conversations, indent=2)};
        
        // Copy the SWEBenchViewer class from the working version
        // ...
    </script>
</body>
</html>'''

def create_readme(conversations, models, timestamp):
    """Create README for the published site."""
    
    return f'''# Your Experiment Name

**Live Demo**: https://yourusername.github.io/your-repo/

## What This Demonstrates

Description of your experiment and what it shows.

### 🔍 Key Features

- **{len(conversations)} Examples**: From {len(models)} different AI models
- **Your Experiment Type**: What to look for in the conversations
- **Real Coding Tasks**: Authentic SWE-Bench problems

### 🤖 Models Featured

{chr(10).join(f'- **{model}**' for model in sorted(models))}

---

*Version {timestamp} - Research on AI model behavior*
'''

if __name__ == "__main__":
    output_dir = Path('publish_your_experiment')
    create_your_experiment_viewer(output_dir)
```

### Step 5: Deploy to GitHub Pages

```bash
# 1. Create your publisher
python3 your_publisher_script.py

# 2. Deploy to GitHub
cd publish_your_experiment
git init
git remote add origin https://github.com/yourusername/your-repo.git
git add .
git commit -m "Add [Your Experiment] evaluation viewer"
git branch -M main
git push -u origin main

# 3. Enable GitHub Pages
# Go to: https://github.com/yourusername/your-repo/settings/pages
# Source: Deploy from branch 'main'
# Folder: / (root)
```

## 🎯 Experiment Types Supported

### Keyword Suppression
Filter condition:
```python
'IMPORTANT: Do not use' in user_message_content
```

### Fact Suppression  
Filter condition:
```python
'Do not state or imply' in user_message_content
```

### Tone Control
Filter condition:
```python
'Use a' in user_message_content and 'tone' in user_message_content
```

### Reasoning Efforts
Filter condition:
```python
conv.get('metadata', {}).get('reasoning_effort') is not None
```

### Custom Experiments
Define your own filter condition based on your experiment design.

## 📁 File Structure

```
your-evaluation-publisher/
├── index.html              # Main viewer (generated)
├── .nojekyll               # Disable Jekyll
├── robots.txt              # Prevent indexing
├── README.md               # Site documentation
└── your_publisher_script.py # Your generation script
```

## 🔧 Customization Options

### Sampling Strategy
```python
# More samples per model
model_sample = random.sample(model_conversations, min(5, len(model_conversations)))

# All conversations (no sampling)
selected_conversations = filtered_conversations

# Specific models only
target_models = ['claude-3-7-sonnet-20250219', 'gpt-4']
filtered_conversations = [c for c in conversations if c['model'] in target_models]
```

### Display Customization
```python
# Custom title and description
title = "My Custom Experiment"
description = "Showing how AI models handle [specific condition]"

# Custom styling (modify CSS)
# Add your custom CSS rules to the style section
```

### Filter Conditions
```python
# Complex conditions
def custom_filter(conv):
    user_msg = next((msg for msg in conv['conversation'] if msg['type'] == 'user_message'), {})
    content = user_msg.get('content', '')
    
    # Multiple conditions
    has_restriction = 'IMPORTANT:' in content
    is_correct = conv.get('is_correct', False)
    model_type = 'claude' in conv.get('model', '')
    
    return has_restriction and is_correct and model_type
```

## 🚨 Common Issues & Solutions

### "Error loading conversations"
- Check that your JSON files are valid
- Ensure the conversation structure matches the expected format
- Verify file paths are correct

### "No conversations match criteria"  
- Check your filter condition
- Print debug info to see what's in your conversations
- Verify the experiment data is in the expected format

### GitHub Pages not updating
- Check that GitHub Pages is enabled in repository settings
- Clear browser cache or use incognito mode
- Wait a few minutes for deployment

### File too large for GitHub Pages
- Reduce `samples_per_model` (try 2 or 1)
- Filter to fewer models
- Truncate very long conversations

## 📊 Performance Guidelines

- **Recommended**: 15-30 conversations total
- **Maximum**: ~50 conversations (to stay under GitHub's 100MB limit)
- **File size target**: Under 5MB for fast loading
- **Models**: 3-8 different models for good comparison

## 🔗 Related Resources

- [SWE-Bench Dataset](https://www.swebench.com/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [Original Research Repository](https://github.com/YuehHanChen/ccot)

---

*This template was created from the successful deployment of keyword suppression experiments. Modify the filter conditions and descriptions for your specific evaluation type.*