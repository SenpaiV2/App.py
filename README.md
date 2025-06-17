from flask import Flask, request, render_template

app = Flask(__name__)

# Example list of colleges (you may need to adjust this to include all relevant entries)
colleges = [
    # Top Tier Colleges (95-99%ile)
    ('COEP Pune - CSE', 98.5, True),  # Trending
    ('VJTI Mumbai - CSE', 98.0, True),  # Trending
    ('COEP Pune - IT', 97.5, False),
    ('DY Patil Pune - CSE', 95.0, False),
    # Add other colleges as needed...
]

TOTAL_CANDIDATES = 1000  # Set this to your actual total candidates

@app.route('/', methods=['GET', 'POST'])
def home():
    if request.method == 'POST':
        mode = request.form['mode']
        category = request.form['category']
        city = request.form['city']
        branches = request.form.getlist('branches')  # Get selected branches
        
        if mode == 'percentile':
            pct = float(request.form['percentile'])
        else:
            rank = int(request.form['rank'])
            pct = max(0, 100 - (rank / TOTAL_CANDIDATES * 100))
        
        # Apply category-based cutoff adjustments
        category_adjustment = {
            'General': 0,
            'EWS': -2,
            'OBC': -5,
            'SC': -10,
            'ST': -15,
            'VJNT': -8,
            'SBC': -6
        }
        
        adjusted_pct = pct + category_adjustment.get(category, 0)
        
        # Filter colleges based on percentile, city, and selected branches
        eligible = []
        for name, cutoff, trending in colleges:
            if adjusted_pct >= cutoff and city.lower() in name.lower():
                # Check if any selected branch matches the college
                branch_match = False
                for branch in branches:
                    if branch == 'CSE' and ('CSE' in name and 'AI' not in name):
                        branch_match = True
                        break
                    elif branch == 'IT' and 'IT' in name:
                        branch_match = True
                        break
                
                if branch_match:
                    eligible.append((name, trending))

        engineering_branches = {
            'CSE': 'Computer Science & Engineering',
            'IT': 'Information Technology',
            # Add other branches if needed...
        }
        
        return render_template('result.html', mode=mode, pct=round(pct, 2), 
                               schools=eligible, category=category, city=city, 
                               branches=branches, engineering_branches=engineering_branches)
    
    return render_template('index.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)