from moviepy.editor import VideoFileClip, CompositeVideoClip, concatenate_videoclips, ImageClip
from PIL import Image, ImageDraw, ImageFont
import numpy as np
import os

# -----------------------------
# 🔹 파일 경로 설정
# -----------------------------
video_path = "시랑 같이.mp4"   # 원본 영상 파일 경로
elder_image_path = "A_digital_photograph_with_overlaid_Korean_text_fea.png"  # 마지막 노인 이미지
output_path = "final_poem_typing_720p.mp4"  # 결과물 저장 경로

# -----------------------------
# 🔹 시 원문
# -----------------------------
poem_lines = [
    "꿈",
    "김주원",
    "기분 좋은 날 이었다",
    "모든 것이 완벽했다",
    "소년은 행복했고",
    "누구보다 들떠 있었다",
    "그 따스한 향기 아래",
    "다시는 돌아오지 못 할",
    "터전 안에서 미소를 머금은채..",
    "소년은 알고 있었다",
    "이게 마지막이라는 것을",
    "그리고 그 곳엔 어느덧",
    "한 노인이 울고 있었다",
]

# -----------------------------
# 🔹 배경 영상 불러오기 & 720p로 리사이즈
# -----------------------------
base_clip = VideoFileClip(video_path).resize(height=720)

# 목표 시간 (20초)
target_duration = 20
line_duration = target_duration / len(poem_lines)

# -----------------------------
# 🔹 폰트 설정 (고딕체)
# -----------------------------
# ⚠️ Windows라면: "C:/Windows/Fonts/malgun.ttf" 로 바꾸세요 (맑은 고딕)
# ⚠️ Mac이라면: "/System/Library/Fonts/AppleSDGothicNeo.ttc"
# ⚠️ Colab/Linux라면: "/usr/share/fonts/truetype/noto/NotoSansCJK-Regular.ttc"
font_path = "/usr/share/fonts/truetype/noto/NotoSansCJK-Regular.ttc"
font_size = 40
font = ImageFont.truetype(font_path, font_size)

# -----------------------------
# 🔹 텍스트 이미지를 PIL로 생성하는 함수
# -----------------------------
def make_text_image(text, size, font):
    img = Image.new("RGBA", size, (0, 0, 0, 0))  # 투명 배경
    draw = ImageDraw.Draw(img)
    w, h = draw.multiline_textsize(text, font=font, spacing=10)
    draw.multiline_text(
        ((size[0]-w)//2, (size[1]-h)//2),  # 중앙 정렬
        text,
        font=font,
        fill=(255, 255, 255, 255),  # 흰색 글씨
        spacing=10,
        align="center"
    )
    return np.array(img)

# -----------------------------
# 🔹 한 줄씩 늘어나는 텍스트 (페이드인 효과)
# -----------------------------
text_clips = []
current_text = ""
for line in poem_lines:
    current_text += line + "\n"
    img = make_text_image(current_text, base_clip.size, font)
    clip = (ImageClip(img)
            .set_duration(line_duration)
            .fadein(0.3))
    text_clips.append(clip)

# 전체 자막 영상
poem_text_clip = concatenate_videoclips(text_clips)

# -----------------------------
# 🔹 배경 + 자막 합성
# -----------------------------
poem_with_video = CompositeVideoClip([base_clip.set_duration(poem_text_clip.duration), poem_text_clip])

# -----------------------------
# 🔹 마지막 노인 이미지 (페이드인 후 3초 여운)
# -----------------------------
if os.path.exists(elder_image_path):
    elder_img = (ImageClip(elder_image_path)
                 .set_duration(3)
                 .fadein(1)
                 .resize(base_clip.size))
    final_clip = concatenate_videoclips([poem_with_video, elder_img])
else:
    final_clip = poem_with_video

# -----------------------------
# 🔹 최종 영상 출력 (720p)
# -----------------------------
final_clip.write_videofile(output_path, codec="libx264", fps=24)

print("✅ 완성! →", output_path)
