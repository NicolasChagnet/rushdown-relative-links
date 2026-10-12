# rushdown-relative-links

This is an extension for the [rushdown](https://crates.io/crates/rushdown) crate.

Rewrite the relative destinations of a document to root-relative ones, resolved against a base directory. A page is usually reachable at more than one URL — `/blog/post` and `/blog/post/` both serve the same file — and a browser resolves `plot.png` against the *document URL*, not against where the file sits. The two spellings then point at different places, and which one a visitor gets depends on the server's redirect policy. Removing that dependency makes the same output work behind `try_files`, behind a server that redirects to the trailing slash, and when the layout changes.

Relative destinations are resolved as *file* references, so a document at `blog/post/index.md` with a sibling `plot.png` refers to `/blog/post/plot.png`. Destinations which need no resolving are left alone: root-relative, protocol-relative, fragment-only, query-only, and anything carrying a scheme (`https:`, `mailto:`, `data:`, …).

## Usage

Add the crate via `cargo add rushdown-relative-links` and include it within your rushdown parser, with the base directory of the document

```rust
use rushdown::new_markdown_to_html;
let parser = new_markdown_to_html(
    Default::default(),
    Default::default(),
    rushdown_relative_links::relative_links_extension("/blog/post"),
    rushdown::renderer::html::empty_renderer_extension(),
);
let mut html = String::new();
parser(&mut html, "![plot](plot.png)").unwrap();
assert!(html.contains(r#"src="/blog/post/plot.png""#));
```

The base directory is a property of the document, not of the parser, so build one extension per document.
