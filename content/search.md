---
title: "Search"
url: "/search/"
---

<script language="javascript">

var archive_results = {};

function runSearch(q) {
    let results_node = document.getElementById("list_results");
    let include_linklog = document.getElementById("include_linklog").checked;

    results_node.innerHTML = "";
    if (q.length > 0) {
        // split query into lowercase words
        let keywords = q.toLowerCase().trim().split(/\s+/);

        for (let i = 0; i < archive_results.items.length; i++) {
            let item = archive_results.items[i];
            
            // --- NEW: Check for Linklog ---
            // Standard JSON Feeds in Hugo output categories into a "tags" array
            let is_linklog = false;
            if (item.tags) {
                is_linklog = item.tags.some(tag => tag.toLowerCase() === "linklog");
            }

            // If the box is NOT checked, and the post IS a linklog, skip to the next item
            if (!include_linklog && is_linklog) {
                continue;
            }
            // ------------------------------

            let title_lower = item.title.toLowerCase();
            let text_lower = item.content_text.toLowerCase();

            // combine title and text into one searchable string
            let full_text = title_lower + " " + text_lower;

            // check if every keyword exists
            let match = keywords.every(function(k) {
                return full_text.includes(k);
            });

            if (match) {
                let p_node = document.createElement("p");        
                let link_node = document.createElement("a");
                let d = Date.parse(item.date_published);
                let date_s = new Date(d).toISOString().substr(0, 10);
                let date_node = document.createTextNode(date_s); 
                link_node.appendChild(date_node);
                link_node.href = item.url;
                
                let title_node = null;
                if (item.title.length > 0) {
                    title_node = document.createElement("span");
                    title_node.innerHTML = ": <b>" + item.title + "</b>"
                }
                
                let s = item.content_text;
                if (s.length > 200) {
                    s = s.substr(0, 200) + "...";
                }
                
                let text_node = document.createElement("span");
                text_node.innerHTML = ": " + s;
                
                p_node.appendChild(link_node);
                if (title_node != null) {
                    p_node.appendChild(title_node);
                }
                p_node.appendChild(text_node);
                results_node.appendChild(p_node);
            }
        }
    }
}

function submitSearch(q) {
    runSearch(q);
    
    // push query and checkbox state into URL
    const url = new URL(window.location.href);
    url.searchParams.set("q", q);
    
    if (document.getElementById("include_linklog").checked) {
        url.searchParams.set("linklog", "true");
    } else {
        url.searchParams.delete("linklog");
    }
    
    history.pushState({}, "", url);
}

document.addEventListener("DOMContentLoaded", function() {
    let loading_timer = setTimeout(function() {
        // show status text if archive doesn't load very quickly
        document.getElementById("list_loading").style.display = "flex";
    }, 1500);

    fetch("/archive/index.json").then(response => response.json()).then(data => {
        archive_results = data;

        clearTimeout(loading_timer);
        document.getElementById("list_loading").style.display = "none";

        // restore from search args
        const url = window.location.href;
        const params = new URLSearchParams(new URL(url).search);
        
        // Restore checkbox state
        if (params.get("linklog") === "true") {
            document.getElementById("include_linklog").checked = true;
        }

        // Restore query string and run search
        const q = params.get("q");
        if (q && (q.length > 0)) {
            document.getElementById("input_search").value = q;
            runSearch(q);
        }
    });
});
    
</script>

<style>

#search {
    display: none;
}

form {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.checkbox-wrapper {
    margin-bottom: 20px;
    font-size: 14px;
    display: flex;
    align-items: center;
    gap: 8px;
}

#list_loading {
    display: none;
    font-size: 14px;
    justify-content: center;
}

.field {
    width: 270px;
    height: 34px;
    font-size: 13px;
    font-weight: 400;
    padding-left: 12px;
    border: 2px solid #eee;
    margin-top: 20px;
    margin-bottom: 10px; /* Reduced to pull checkbox closer */
    border-radius: 17px;
    -webkit-appearance: none;
}

</style>

<form onSubmit="return false;">
    <input class="field" type="text" name="q" id="input_search" placeholder="Search" onChange="submitSearch(this.value.toLowerCase());">    
    <div class="checkbox-wrapper">
        <!-- Re-run search whenever the box is ticked/unticked -->
        <input type="checkbox" id="include_linklog" onChange="submitSearch(document.getElementById('input_search').value.toLowerCase());">
        <label for="include_linklog">Include Linklog posts</label>
    </div>
</form>

<div id="list_loading">
    Loading posts...
</div>

<div id="list_results">
</div>
