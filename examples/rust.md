# Rust

In this scenario we are assuming you have a folder structure that looks like this:

```bash
.
├── Cargo.lock
├── Cargo.toml
├── src
│   ├── app.rs
│   ├── main.rs
│   ├── theme.rs
│   └── ui
```

Another thing we are assuming in this scenario is that you are going to need all the colors, when in reality most people can stick to using only semantic colors (like fg, bg, ...) if you do have illustrations this is where you would like to have something like a palette.

## Getting themes

In order to install all the themes into `~/.config/lotus/`:

```bash
mkdir -p ~/.config/lotus
cd ~/.config/lotus
git clone --filter=blob:none --no-checkout https://github.com/lotus-io/lotus.git .
git sparse-checkout init --no-cone
git sparse-checkout set 'themes/**'
git checkout
rm -rf .git
```

## Rust

In order to be able to interact with the themes I suggest having something like `src/themes.rs` with the folowwing helpers:

```rust
use ratatui::style::Color;
use serde::{Deserialize, Deserializer};

#[derive(Debug, Clone, Deserialize)]
pub struct Theme {
    pub name: String,
    pub author: String,
    pub colors: ThemeColors,
}

#[derive(Debug, Clone, Deserialize)]
pub struct ThemeColors {
    #[serde(deserialize_with = "deserialize_color")]
    pub fg: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub bg: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub surface: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub surface_2: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub text: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub muted: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub subtle: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub disabled: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub border: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub selection_fg: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub selection_bg: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub highlight_fg: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub highlight_bg: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub accent: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub success: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub warning: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub error: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub info: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub link: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub black: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub blue: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub green: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub cyan: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub red: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub magenta: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub brown: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub light_gray: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub dark_gray: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub light_blue: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub light_green: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub light_cyan: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub light_red: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub light_magenta: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub yellow: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub white: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub orange: Color,
    #[serde(deserialize_with = "deserialize_color")]
    pub pink: Color,
}

impl Theme {
    pub fn fallback() -> Self {
        Self {
            name: format!("{}", env!("CARGO_PKG_NAME")).to_string(),
            author: format!("{}", env!("CARGO_PKG_VERSION")).to_string(),
            version: 1,
            colors: ThemeColors {
                // here your local theme so you can use it as a fallback
                fg: Color::Black,
                bg: Color::Black,
                surface: Color::Black,
                surface_2: Color::Black,
                text: Color::Black,
                muted: Color::Black,
                subtle: Color::Black,
                disabled: Color::Black,
                border: Color::Black,
                selection_fg: Color::Black,
                selection_bg: Color::Black,
                highlight_fg: Color::Black,
                highlight_bg: Color::Black,
                accent: Color::Black,
                success: Color::Black,
                warning: Color::Black,
                error: Color::Black,
                info: Color::Black,
                link: Color::Black,
                black: Color::Black,
                blue: Color::Black,
                green: Color::Black,
                cyan: Color::Black,
                red: Color::Black,
                magenta: Color::Black,
                brown: Color::Black,
                light_gray: Color::Black,
                dark_gray: Color::Black,
                light_blue: Color::Black,
                light_green: Color::Black,
                light_cyan: Color::Black,
                light_red: Color::Black,
                light_magenta: Color::Black,
                yellow: Color::Black,
                white: Color::Black,
                orange: Color::Black,
                pink: Color::Black,
            },
        }
    }
}

pub fn parse_hex_color(raw: &str) -> Option<Color> {
    let hex = raw.trim().trim_start_matches('#');

    if hex.len() != 6 {
        return None;
    }

    let r = u8::from_str_radix(&hex[0..2], 16).ok()?;
    let g = u8::from_str_radix(&hex[2..4], 16).ok()?;
    let b = u8::from_str_radix(&hex[4..6], 16).ok()?;

    Some(Color::Rgb(r, g, b))
}

fn deserialize_color<'de, D>(deserializer: D) -> Result<Color, D::Error>
where
    D: Deserializer<'de>,
{
    let raw = String::deserialize(deserializer)?;

    parse_hex_color(&raw)
        .ok_or_else(|| serde::de::Error::custom(format!("invalid hex color: {raw:?}")))
}

fn parse_theme_toml(raw: &str) -> Result<Theme, toml::de::Error> {
    toml::from_str(raw)
}

fn themes_dir() -> Option<std::path::PathBuf> {
    let home = std::env::var_os("HOME")?;
    Some(std::path::PathBuf::from(home).join(".config/lotus/themes"))
}

fn load_themes() -> Vec<Theme> {
    let Some(dir) = themes_dir() else {
        return Vec::new();
    };

    let Ok(entries) = std::fs::read_dir(&dir) else {
        return Vec::new();
    };

    entries
        .filter_map(Result::ok)
        .filter(|entry| entry.path().extension().is_some_and(|ext| ext == "toml"))
        .filter_map(|entry| match std::fs::read_to_string(entry.path()) {
            Ok(raw) => match parse_theme_toml(&raw) {
                Ok(theme) => Some(theme),
                Err(err) => {
                    eprintln!("Failed to parse theme {}: {err}", entry.path().display());
                    None
                }
            },
            Err(err) => {
                eprintln!("Failed to read theme {}: {err}", entry.path().display());
                None
            }
        })
        .collect()
}

pub fn discover_themes() -> Vec<Theme> {
    let mut themes = load_themes();

    themes.sort_by(|a, b| a.name.cmp(&b.name));

    if themes.is_empty() {
        themes.push(Theme::fallback());
    }

    themes
}

pub fn list_themes() {}

pub fn get_current_theme() {}

pub fn set_current_theme() {}
```