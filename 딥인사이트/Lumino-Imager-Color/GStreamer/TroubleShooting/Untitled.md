```
{python icon title:foo} print("This is inline code")
```

#!/usr/bin/env python3

"""

Lumino-Imager-Color 카메라 애플리케이션

  

비디오 캡처와 녹화를 위한 메인 애플리케이션입니다.

"""

  

# 라즈베리파이 GPIO 관련 오류 방지를 위해 mock_gpio 모듈 먼저 임포트

import sys

import os

  

# 현재 디렉토리를 sys.path에 추가하여 core 패키지를 임포트할 수 있도록 함

sys.path.append(os.path.dirname(os.path.abspath(__file__)))

  

# JYP

# GPIO 모듈 가상화

#try:

# from core.mock_gpio import MockLGPIO, MockRPiGPIO

# 이 시점에서 가상 GPIO 모듈은 이미 sys.modules에 등록되어 있음

#except ImportError:

# print("Warning: mock_gpio 모듈을 로드할 수 없습니다. GPIO 관련 오류가 발생할 수 있습니다.")

  

# 이제 나머지 import 진행

import time

import signal

import threading

import traceback

import logging

from typing import Dict, List, Any, Optional, Union, Callable, Tuple

  

# Global variables

frame_count = 0

skip_frames = 1

  

# GStreamer 시간 상수 (나노초 단위)

GST_MILLISECOND = 1000000 # 1밀리초 = 10^6 나노초

  

# 로깅 설정

logging.basicConfig(

level=logging.INFO,

format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'

)

logger = logging.getLogger("CameraApp")

logger.info("Camera App starting...")

  

# GStreamer 임포트 시도

try:

import gi

gi.require_version('Gst', '1.0')

from gi.repository import Gst, GLib

  

# GStreamer 초기화

Gst.init(None)

GSTREAMER_AVAILABLE = True

except (ImportError, ValueError) as e:

logger.warning(f"Failed to import GStreamer: {e}")

GSTREAMER_AVAILABLE = False

  

# 멀티스레드 아키텍처 사용 여부

USE_MULTITHREADED_ARCHITECTURE = True

  

class GStreamerWrapper:

"""GStreamer 관련 기능을 캡슐화한 래퍼 클래스"""

def __init__(self):

"""GStreamer 초기화"""

if GSTREAMER_AVAILABLE:

Gst.init(None)

  

def create_element(self, factory_name: str, element_name: str) -> Optional[Any]:

"""GStreamer 엘리먼트 생성"""

if not GSTREAMER_AVAILABLE:

logger.warning(f"GStreamer not available, cannot create element: {factory_name}")

return None

return Gst.ElementFactory.make(factory_name, element_name)

def create_pipeline(self, name: str) -> Optional[Any]:

"""GStreamer 파이프라인 생성"""

if not GSTREAMER_AVAILABLE:

logger.warning(f"GStreamer not available, cannot create pipeline: {name}")

return None

return Gst.Pipeline.new(name)

def create_caps(self, caps_string: str) -> Optional[Any]:

"""GStreamer Caps 생성"""

if not GSTREAMER_AVAILABLE:

logger.warning(f"GStreamer not available, cannot create caps: {caps_string}")

return None

return Gst.Caps.from_string(caps_string)

def create_main_loop(self) -> Optional[Any]:

"""GLib 메인 루프 생성"""

if not GSTREAMER_AVAILABLE:

logger.warning("GStreamer not available, cannot create GLib.MainLoop")

return None

return GLib.MainLoop()

def get_state_enum(self, state_name: str) -> Optional[Any]:

"""상태 열거형 가져오기"""

if not GSTREAMER_AVAILABLE:

logger.warning(f"GStreamer not available, cannot get state enum: {state_name}")

return None

if state_name == "PLAYING":

return Gst.State.PLAYING

elif state_name == "PAUSED":

return Gst.State.PAUSED

elif state_name == "NULL":

return Gst.State.NULL

else:

logger.error(f"Unknown state name: {state_name}")

return None

def create_eos_event(self) -> Optional[Any]:

"""EOS 이벤트 생성"""

if not GSTREAMER_AVAILABLE:

logger.warning("GStreamer not available, cannot create EOS event")

return None

return Gst.Event.new_eos()

def get_message_type_enum(self, type_name: str) -> Optional[Any]:

"""메시지 타입 열거형 가져오기"""

if not GSTREAMER_AVAILABLE:

logger.warning(f"GStreamer not available, cannot get message type enum: {type_name}")

return None

if type_name == "EOS":

return Gst.MessageType.EOS

elif type_name == "ERROR":

return Gst.MessageType.ERROR

else:

logger.error(f"Unknown message type name: {type_name}")

return None

def state_change_return_get_name(self, state_ret: Any) -> str:

"""상태 변경 반환값 이름 가져오기"""

if not GSTREAMER_AVAILABLE:

return "UNKNOWN"

if state_ret == Gst.StateChangeReturn.SUCCESS:

return "SUCCESS"

elif state_ret == Gst.StateChangeReturn.ASYNC:

return "ASYNC"

elif state_ret == Gst.StateChangeReturn.NO_PREROLL:

return "NO_PREROLL"

elif state_ret == Gst.StateChangeReturn.FAILURE:

return "FAILURE"

else:

return f"UNKNOWN({state_ret})"

def wait_for_state_change(self, element: Any, timeout_ns: int = Gst.CLOCK_TIME_NONE) -> Optional[Gst.StateChangeReturn]:

"""상태 변경 완료 대기"""

if not GSTREAMER_AVAILABLE or not element:

return None

return element.get_state(timeout_ns)[0]

  
  

class VideoRecorder:

def __init__(self, gst_wrapper: Optional[GStreamerWrapper] = None,

device: str = "/dev/video0",

output_pattern: str = None,

width: int = 1280,

height: int = 720,

framerate: int = 30,

bitrate: int = 5000):

"""

비디오 레코더 초기화

Args:

gst_wrapper: GStreamer 래퍼 객체 (테스트용 목업 주입 가능)

device: 카메라 장치 경로

output_pattern: 출력 파일 패턴 (None일 경우 타임스탬프 사용)

width: 영상 너비

height: 영상 높이

framerate: 프레임레이트

bitrate: 비트레이트 (kbps)

"""

# 의존성 주입 (테스트용)

self.gst = gst_wrapper if gst_wrapper is not None else GStreamerWrapper()

# 설정 저장

self.device = device

# 타임스탬프 기반 출력 패턴 생성

if output_pattern is None:

from datetime import datetime

timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")

self.output_pattern = f"video_{timestamp}_%03d.mp4"

logger.info(f"Using timestamp-based output pattern: {self.output_pattern}")

else:

self.output_pattern = output_pattern

self.width = width

self.height = height

self.framerate = framerate

self.bitrate = bitrate

# 내부 상태 변수

self.pipeline = None

self.source = None

self.jpeg_filter = None

self.jpegdec = None

self.converter = None

self.raw_fps_filter = None

self.raw_fmt_filter = None

self.valve = None # valve 요소 추가

self.encoder = None

self.parser = None

self.splitmuxsink = None

self.muxer = None # muxer 객체 참조 저장

self.loop = None

self.loop_thread = None

self.is_running = False

self.received_eos = False # EOS 이벤트 수신 여부 추적

# 녹화 경로 동적 연결을 위한 패드 참조 저장

self.tee_record_pad = None # tee에서 녹화 경로로 가는 패드

self.queue2_sink_pad = None # 녹화 큐의 싱크 패드

self.is_recording = False # 현재 녹화 중인지 여부

# 스냅샷 관련 요소들 추가

self.tee = None # 비디오 스트림 분기 tee

self.snapshot_queue = None # 스냅샷용 큐

self.snapshot_convert = None # 스냅샷 포맷 변환

self.snapshot_jpegenc = None # JPEG 인코더

self.snapshot_sink = None # 파일 싱크

self.snapshot_caps = None # 스냅샷 캡스 필터

# 스냅샷 관련 상태 변수

self.snapshot_pending = False

self.snapshot_filename = None

# 타임스탬프 관리를 위한 상태 변수

self.current_segment = 0 # 현재 파일 세그먼트

self.base_time = 0 # 현재 세그먼트 시작 기준 시간

self.segment_start_time = 0 # 세그먼트 시작 시간

# 초기화 메서드 호출 - 제거함 (start 메서드에서 호출됨)

# self._initialize_pipeline()

  

def do_handoff(self, identity, buffer, *args):

"""

GStreamer identity 엘리먼트의 handoff 시그널 콜백

처음 N개의 프레임을 건너뛰기 위해 사용

Args:

identity: identity 엘리먼트

buffer: 현재 버퍼

*args: 추가 인수 (GStreamer 버전에 따라 다를 수 있음)

Returns:

bool: 프레임 유지 여부 (True: 유지, False: 드롭)

"""

global frame_count

frame_count += 1

if frame_count <= skip_frames:

logger.info(f"[DROP] Frame {frame_count}")

self.valve.set_property("drop", True)

#buffer.set_size(0)

else:

self.valve.set_property("drop", False)

def _initialize_pipeline(self) -> bool:

"""

GStreamer 파이프라인 초기화

Returns:

bool: 초기화 성공 여부

"""

if not GSTREAMER_AVAILABLE:

logger.warning("GStreamer not available, pipeline initialization skipped")

return False

try:

# 첫 프레임 스킵 설정 로깅

global skip_frames

logger.info(f"첫 {skip_frames}개 프레임 스킵 설정 (First {skip_frames} frames will be skipped)")

# 파이프라인 생성

self.pipeline = self.gst.create_pipeline("video-recorder")

  

# Elements 생성

self.source = self.gst.create_element("v4l2src", "source")

self.identity = self.gst.create_element("identity", "identity")

self.valve = self.gst.create_element("valve", "valve")

self.videorate = self.gst.create_element("videorate", "videorate")

self.jpeg_filter = self.gst.create_element("capsfilter", "jpeg_filter")

self.jpegdec = self.gst.create_element("jpegdec", "jpeg-decoder")

self.converter = self.gst.create_element("v4l2convert", "converter")

self.raw_fps_filter = self.gst.create_element("capsfilter", "raw_fps_filter")

self.raw_fmt_filter = self.gst.create_element("capsfilter", "raw_fmt_filter")

# Tee 요소 추가 (비디오 스트림 분기)

self.tee = self.gst.create_element("tee", "stream_tee")

# 녹화 경로 요소

self.encoder = self.gst.create_element("x264enc", "encoder")

self.parser = self.gst.create_element("h264parse", "parser")

self.splitmuxsink = self.gst.create_element("splitmuxsink", "sink")

# 스냅샷 경로 요소

self.snapshot_queue = self.gst.create_element("queue", "snapshot_queue")

self.snapshot_convert = self.gst.create_element("videoconvert", "snapshot_convert")

self.snapshot_caps = self.gst.create_element("capsfilter", "snapshot_caps")

self.snapshot_jpegenc = self.gst.create_element("jpegenc", "snapshot_jpegenc")

self.snapshot_sink = self.gst.create_element("fakesink", "snapshot_sink") # 처음에는 fakesink로 설정

# Queue 요소 생성 (버퍼링 및 스레드 분리)

self.queue0 = self.gst.create_element("queue", "queue0")

self.queue1 = self.gst.create_element("queue", "queue1")

self.queue2 = self.gst.create_element("queue", "queue2")

# 모든 요소 생성 확인

elements = [

self.source, self.identity, self.valve, self.videorate,

self.jpeg_filter, self.jpegdec, self.converter,

self.raw_fps_filter, self.raw_fmt_filter, self.tee,

self.encoder, self.parser, self.splitmuxsink,

self.snapshot_queue, self.snapshot_convert,

self.snapshot_caps, self.snapshot_jpegenc, self.snapshot_sink,

self.queue0, self.queue1, self.queue2

]

  

if not all(elements):

logger.error("Failed to create one or more GStreamer elements")

return False

return self._configure_pipeline(elements)

except Exception as e:

logger.error(f"Pipeline initialization error: {e}")

return False

def _configure_pipeline(self, elements: List[Any]) -> bool:

"""

파이프라인 요소 설정 및 연결

Args:

elements: 설정할 GStreamer 요소 목록

Returns:

bool: 설정 성공 여부

"""

try:

# 속성 설정

self.source.set_property("device", self.device)

self.source.set_property("io-mode", "dmabuf")

self.source.set_property("do-timestamp", True)

  

# valve 설정

self.valve.set_property("drop", False) # 초기값 False로 설정

# identity 설정

self.identity.connect("handoff", self.do_handoff)

# 필수 설정: 시그널 활성화 및 버퍼 제어 옵션

self.identity.set_property("signal-handoffs", True) # handoff 시그널 활성화 (필수)

self.identity.set_property("drop-allocation", True) # 버퍼 할당 드롭 허용

self.identity.set_property("silent", False) # 디버그용 로그 출력 활성화

# 사용하지 않는 속성들 (참고용)

#self.identity.set_property("sync", True)

#self.identity.set_property("datarate", 0)

#self.identity.set_property("drop-buffer-flags", Gst.BufferFlags.CORRUPTED)

  

# videorate 설정

#self.videorate.set_property("skip-to-first", True)

#self.videorate.set_property("drop-only", True)

#self.videorate.set_property("max-rate", 30)

# Queue 설정

# 소스와 변환기 사이의 큐

self.queue0.set_property("max-size-buffers", 30)

self.queue0.set_property("max-size-bytes", 0) # 무제한

self.queue0.set_property("max-size-time", 0) # 무제한

self.queue0.set_property("leaky", 2) # downstream (오래된 버퍼 드롭)

# JPEG 디코더와 변환기 사이의 큐

self.queue1.set_property("max-size-buffers", 30)

self.queue1.set_property("max-size-bytes", 0)

self.queue1.set_property("max-size-time", 0)

# 인코더와 파서 사이의 큐

self.queue2.set_property("max-size-buffers", 30)

self.queue2.set_property("max-size-bytes", 0)

self.queue2.set_property("max-size-time", 0)

jpeg_caps = self.gst.create_caps(

f"image/jpeg,width={self.width},height={self.height},framerate={self.framerate}/1")

self.jpeg_filter.set_property("caps", jpeg_caps)

raw_caps = self.gst.create_caps("video/x-raw,framerate=30/1")

self.raw_fps_filter.set_property("caps", raw_caps)

  

raw_caps = self.gst.create_caps("video/x-raw,format=NV12")

self.raw_fmt_filter.set_property("caps", raw_caps)

# 인코더 설정 개선 - 처음 프레임 품질 향상

self.encoder.set_property("bitrate", self.bitrate)

self.encoder.set_property("speed-preset", "ultrafast")

self.encoder.set_property("tune", "zerolatency")

#self.encoder.set_property("key-int-max", 30) # 키프레임 간격 (fps 기준)

# 첫 번째 프레임은 항상 키프레임(I-프레임)으로 설정

#self.encoder.set_property("insert-vui", True) # VUI 헤더 삽입으로 디코더 호환성 향상

#self.encoder.set_property("option-string", "keyint=30:min-keyint=1:vbv-init=1:rc-lookahead=0:ref=1:bframes=0:scenecut=0:force-cfr=1")

# qp 또는 quantizer 값을 낮게 설정하여 초기 프레임의 품질 향상

self.encoder.set_property("qp-min", 10) # 최소 양자화 파라미터 (낮을수록 고품질)

# 파서 설정 추가

#self.parser.set_property("config-interval", 1) # SPS/PPS를 각 I-프레임마다 삽입

# Muxer 설정

self.muxer = self.gst.create_element("mp4mux", "muxer")

if not self.muxer:

logger.error("Failed to create mp4mux element")

return False

self.muxer.set_property("faststart", True)

self.muxer.set_property("fragment-duration", 1000) # 1초마다 파일 업데이트

# movie-timescale을 설정하여 시간 정확도 개선

self.muxer.set_property("movie-timescale", 90000) # 90kHz 타임스케일 (표준 사용)

# 출력 설정

self.splitmuxsink.connect("format-location", self.on_format_location)

#self.splitmuxsink.set_property("location", self.output_pattern)

self.splitmuxsink.set_property("max-size-time", 60 * Gst.SECOND) # 10분(600초)

self.splitmuxsink.set_property("muxer", self.muxer)

# 새 파일마다 muxer 초기화

self.splitmuxsink.set_property("reset-muxer", True)

# splitmuxsink 설정 최적화

self.splitmuxsink.set_property("async-handling", False) # 비동기 처리 활성화

self.splitmuxsink.set_property("send-keyframe-requests", True) # 파일 분할 전 키프레임 요청

# 추가 설정: 파일 저장 개선

# max-size-bytes를 0으로 설정하여 크기 제한 없이 저장

self.splitmuxsink.set_property("max-size-bytes", 0)

# 모든 버퍼를 즉시 디스크에 기록

self.splitmuxsink.set_property("async-finalize", True)

# 스플릿 시간 정책 설정

self.splitmuxsink.set_property("muxer-factory", "mp4mux")

# 스냅샷 요소 설정

# 스냅샷 큐 설정

self.snapshot_queue.set_property("max-size-buffers", 2) # 버퍼 갯수 제한

self.snapshot_queue.set_property("leaky", 2) # 오래된 버퍼 버림(downstream)

# 스냅샷 캡스 필터 설정 (RGB 포맷으로 변환)

snapshot_caps_str = f"video/x-raw,format=RGB,width={self.width},height={self.height}"

snapshot_caps = self.gst.create_caps(snapshot_caps_str)

self.snapshot_caps.set_property("caps", snapshot_caps)

# JPEG 인코딩 품질 설정

self.snapshot_jpegenc.set_property("quality", 85) # 0-100, 높을수록 좋은 품질

# 스냅샷 싱크 설정

self.snapshot_sink.set_property("sync", False) # 동기화 비활성화

self.snapshot_sink.set_property("async", False) # 비동기 처리 비활성화

self.snapshot_sink.set_property("signal-handoffs", True) # 버퍼 핸드오프 시그널 활성화

self.snapshot_sink.connect("handoff", self._on_snapshot_handoff) # 핸드오프 콜백 연결

  

# 파이프라인에 추가

for elem in elements:

self.pipeline.add(elem)

  

# Elements 링크

if not self._link_elements():

logger.error("Failed to link pipeline elements")

return False

# 인코더 출력에 패드 프로브 설치

#encoder_src_pad = self.encoder.get_static_pad("src")

#if encoder_src_pad:

# probe_id = encoder_src_pad.add_probe(

# Gst.PadProbeType.BUFFER,

# self.timestamp_probe_callback,

# None

# )

# logger.info(f"타임스탬프 관리 프로브 설정 완료: ID={probe_id}")

#else:

# logger.error("인코더 출력 패드를 찾을 수 없음")

# return False

return True

except Exception as e:

logger.error(f"Pipeline configuration error: {e}")

logger.error(traceback.format_exc())

return False

def _link_elements(self) -> bool:

"""

파이프라인 요소들을 연결

Returns:

bool: 연결 성공 여부

"""

try:

# 메인 경로 연결 (소스 -> tee 부분)

self.source.link(self.queue0)

self.queue0.link(self.jpeg_filter)

self.jpeg_filter.link(self.jpegdec)

self.jpegdec.link(self.queue1)

self.queue1.link(self.converter)

self.converter.link(self.videorate)

self.videorate.link(self.raw_fps_filter)

self.raw_fps_filter.link(self.identity)

self.identity.link(self.valve)

self.valve.link(self.raw_fmt_filter)

self.raw_fmt_filter.link(self.tee)

# 녹화 경로 패드 준비 (시작 시에는 연결하지 않음)

# tee의 src 패드에서 request 패드를 요청

self.tee_record_pad = self.tee.get_request_pad("src_%u")

self.queue2_sink_pad = self.queue2.get_static_pad("sink")

if not self.tee_record_pad or not self.queue2_sink_pad:

logger.error("Failed to get pads for recording branch")

return False

# 주의: 여기서는 실제로 링크하지 않음, 녹화 시작 시 동적으로 링크됨

logger.info("녹화 경로 패드 준비 완료 (연결은 녹화 시작 시에 수행됨)")

# 나머지 녹화 경로 요소들은 미리 연결해둠

self.queue2.link(self.encoder)

self.encoder.link(self.parser)

self.parser.link(self.splitmuxsink)

# 스냅샷 경로 연결 (tee -> fakesink 부분)

# tee의 src 패드에서 다른 request 패드를 요청하여 snapshot_queue에 연결

tee_snapshot_pad = self.tee.get_request_pad("src_%u")

snapshot_queue_sink_pad = self.snapshot_queue.get_static_pad("sink")

if tee_snapshot_pad and snapshot_queue_sink_pad:

tee_snapshot_pad.link(snapshot_queue_sink_pad)

else:

logger.error("Failed to get and link tee pads for snapshot branch")

return False

# 나머지 스냅샷 경로 연결

self.snapshot_queue.link(self.snapshot_convert)

self.snapshot_convert.link(self.snapshot_caps)

self.snapshot_caps.link(self.snapshot_jpegenc)

self.snapshot_jpegenc.link(self.snapshot_sink)

logger.info("파이프라인 요소들이 성공적으로 연결되었습니다. 녹화 경로는 준비 상태입니다.")

return True

except Exception as e:

logger.error(f"Error linking pipeline elements: {e}")

logger.error(traceback.format_exc())

return False

def _on_snapshot_handoff(self, sink, buffer, pad):

"""

스냅샷 버퍼 핸드오프 콜백

스냅샷 요청이 있을 때 호출되는 콜백 함수로,

JPEG 인코딩된 이미지 버퍼를 파일로 저장합니다.

Args:

sink: 스냅샷 싱크 요소

buffer: 이미지 버퍼

pad: 싱크 패드

"""

try:

if not self.snapshot_pending or not self.snapshot_filename:

return

# 버퍼에서 데이터 추출

sample = Gst.Sample.new(buffer, None, None, None)

if not sample:

logger.error("스냅샷 샘플을 생성할 수 없습니다")

return

buffer = sample.get_buffer()

if not buffer:

logger.error("스냅샷 버퍼를 가져올 수 없습니다")

return

# 버퍼 맵핑

success, map_info = buffer.map(Gst.MapFlags.READ)

if not success:

logger.error("스냅샷 버퍼를 맵핑할 수 없습니다")

return

try:

# 파일로 저장

with open(self.snapshot_filename, 'wb') as f:

f.write(map_info.data)

logger.info(f"스냅샷이 {self.snapshot_filename}에 저장되었습니다")

finally:

# 버퍼 언맵핑

buffer.unmap(map_info)

# 스냅샷 요청 완료

self.snapshot_pending = False

self.snapshot_filename = None

except Exception as e:

logger.error(f"스냅샷 저장 중 오류 발생: {e}")

logger.error(traceback.format_exc())

self.snapshot_pending = False

  

def bus_call(self, bus: Any, message: Any, loop: Any) -> bool:

"""

GStreamer 버스 메시지 처리 콜백

Args:

bus: GStreamer 버스 객체

message: 버스 메시지

loop: GLib 메인 루프

Returns:

bool: 계속 처리할지 여부

"""

try:

t = message.type

eos_type = self.gst.get_message_type_enum("EOS")

error_type = self.gst.get_message_type_enum("ERROR")

if t == eos_type:

logger.info("Received EOS from pipeline")

self.received_eos = True

# splitmuxsink가 파일을 완료할 수 있도록 충분한 시간 제공

threading.Timer(2.0, self._delayed_quit_loop).start()

return True

elif t == error_type:

err, debug = message.parse_error()

logger.error(f"Error: {err}: {debug}")

if debug:

logger.debug(f"Error details: {debug}")

if loop and hasattr(loop, 'quit'):

loop.quit()

self.is_running = False

# 파일 분할 이벤트 감지 (element message 확인)

elif t == Gst.MessageType.ELEMENT:

structure = message.get_structure()

if structure:

structure_name = structure.get_name()

if "splitmuxsink" in structure_name:

# splitmuxsink 관련 이벤트 처리

try:

fragment_id = None

if structure.has_field("fragment-id"):

fragment_id = structure.get_value("fragment-id")

if "fragment-closed" in structure_name:

logger.info(f"파일 분할 완료: fragment-id={fragment_id}")

elif "fragment-opened" in structure_name:

# 새 파일이 열릴 때 타임스탬프 관련 로그 출력

logger.info(f"새 파일 시작: fragment-id={fragment_id}, 패드 프로브에서 타임스탬프 조정됨")

# 참고: 실제 타임스탬프 조정은 패드 프로브에서 수행

except Exception as e:

logger.error(f"Error processing splitmuxsink event: {e}")

except Exception as e:

logger.error(f"Error in bus_call: {e}")

logger.error(traceback.format_exc())

self.is_running = False

return True

  

def _delayed_quit_loop(self) -> None:

"""지연된 루프 종료 (muxer가 파일 작성을 완료할 시간을 줌)"""

logger.info("Executing delayed loop quit")

# 파이프라인 상태를 PAUSED로 변경하여 버퍼 플러시 유도

if self.pipeline:

try:

paused_state = self.gst.get_state_enum("PAUSED")

if paused_state:

self.pipeline.set_state(paused_state)

# 잠시 대기

time.sleep(1.0)

except Exception as e:

logger.error(f"Error in delayed quit: {e}")

# 루프 종료

if self.loop and hasattr(self.loop, 'quit'):

self.loop.quit()

self.is_running = False

def loop_run_thread(self) -> None:

"""별도의 쓰레드에서 GLib 메인 루프를 실행"""

try:

if self.loop and hasattr(self.loop, 'run'):

self.loop.run()

except Exception as e:

logger.error(f"Error in GLib MainLoop thread: {e}")

finally:

self.is_running = False

logger.info("MainLoop thread exited")

def on_format_location(self, splitmux, fragment_id):

from datetime import datetime

timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")

new_filename = f"video_{timestamp}.mp4"

logger.info(f"[splitmuxsink]...: {new_filename}")

return new_filename

  

def start(self) -> bool:

"""

카메라 스트리밍 시작 (녹화 없이 미리보기만 시작)

Returns:

bool: 시작 성공 여부

"""

if not GSTREAMER_AVAILABLE:

logger.error("GStreamer not available, cannot start recording")

return False

if self.is_running:

logger.warning("Streaming is already running")

return True

try:

# 프레임 카운터 초기화

global frame_count

frame_count = 0

logger.info(f"프레임 카운터 초기화 완료 (Frame counter reset)")

# 타임스탬프 상태 변수 초기화

self.current_segment = 0

self.base_time = 0

self.segment_start_time = 0

logger.info("타임스탬프 상태 변수 초기화 완료")

# 파이프라인이 이미 존재하면 정리 및 재생성

if self.pipeline:

logger.info("Cleaning up existing pipeline before starting new one")

null_state = self.gst.get_state_enum("NULL")

if null_state:

self.pipeline.set_state(null_state)

self.pipeline = None

# 매번 새로운 파이프라인 생성 (명시적으로 초기화 메서드 호출)

if not self._initialize_pipeline():

logger.error("Failed to initialize pipeline")

return False

# 파이프라인 상태 변경 - PAUSED 상태로 먼저 전환하여 버퍼 채우기

paused_state = self.gst.get_state_enum("PAUSED")

if paused_state:

logger.info("Setting pipeline to PAUSED state to preroll")

ret = self.pipeline.set_state(paused_state)

logger.info(f"Pipeline state change to PAUSED returned: {self.gst.state_change_return_get_name(ret)}")

# 버퍼링을 위해 잠시 대기

time.sleep(0.5)

# preroll 상태 확인

if ret == Gst.StateChangeReturn.ASYNC:

self.gst.wait_for_state_change(self.pipeline, 3 * Gst.SECOND)

# 이제 PLAYING 상태로 전환

playing_state = self.gst.get_state_enum("PLAYING")

if not playing_state:

logger.error("Failed to get PLAYING state enum")

return False

ret = self.pipeline.set_state(playing_state)

state_ret_name = self.gst.state_change_return_get_name(ret)

logger.info(f"Pipeline state change to PLAYING returned: {state_ret_name}")

if ret == Gst.StateChangeReturn.FAILURE:

logger.error("Failed to set pipeline to PLAYING state")

return False

# 버스 설정

bus = self.pipeline.get_bus()

bus.add_signal_watch()

self.loop = self.gst.create_main_loop()

if not self.loop:

logger.error("Failed to create GLib.MainLoop")

return False

bus.connect("message", self.bus_call, self.loop)

logger.info("Camera streaming started (without recording)")

# 별도의 쓰레드에서 메인 루프 실행

self.is_running = True

self.is_recording = False # 녹화 상태 초기화

self.received_eos = False # EOS 플래그 초기화

self.loop_thread = threading.Thread(target=self.loop_run_thread)

self.loop_thread.daemon = True

self.loop_thread.start()

return True

except Exception as e:

logger.error(f"Error starting streaming: {e}")

self.is_running = False

return False

  

def start_recording(self) -> bool:

"""

녹화 시작 - tee와 녹화 큐 사이를 동적으로 연결

Returns:

bool: 녹화 시작 성공 여부

"""

if not self.is_running:

logger.error("파이프라인이 실행 중이 아닙니다. 녹화를 시작할 수 없습니다.")

return False

if self.is_recording:

logger.warning("녹화가 이미 진행 중입니다.")

return True

try:

logger.info("녹화 시작: tee와 녹화 경로를 연결합니다.")

# 녹화 시작 시 타임스탬프 기반 출력 패턴 업데이트

#from datetime import datetime

#timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")

#updated_pattern = f"video_{timestamp}_%03d.mp4"

#self.splitmuxsink.set_property("location", updated_pattern)

#logger.info(f"Recording with output pattern: {updated_pattern}")

# tee와 녹화 큐 사이 패드 연결

if self.tee_record_pad and self.queue2_sink_pad:

# 패드가 이미 연결되어 있는지 확인 (중복 연결 방지)

if self.tee_record_pad.is_linked():

logger.warning("녹화 경로 패드가 이미 연결되어 있습니다.")

self.is_recording = True

return True

# 1. 옵셔널: 연결 전에 패드 활성화 상태 확인

if not self.tee_record_pad.is_active():

logger.warning("tee_record_pad가 활성화되지 않았습니다. 활성화를 시도합니다.")

self.tee_record_pad.set_active(True)

if not self.queue2_sink_pad.is_active():

logger.warning("queue2_sink_pad가 활성화되지 않았습니다. 활성화를 시도합니다.")

self.queue2_sink_pad.set_active(True)

# 2. 패드 연결 시도 전 타입 호환성 확인

src_caps = self.tee_record_pad.get_current_caps()

sink_caps = self.queue2_sink_pad.get_current_caps()

if src_caps and sink_caps:

logger.info(f"소스 패드 캡스: {src_caps.to_string()}")

logger.info(f"싱크 패드 캡스: {sink_caps.to_string()}")

# 캡스 호환성 확인 (옵셔널)

if not src_caps.is_subset(sink_caps) and not sink_caps.is_subset(src_caps):

logger.warning("패드 캡스가 완전히 호환되지 않을 수 있지만 연결을 시도합니다.")

# 3. 패드 연결 시도

logger.info("패드 연결을 시도합니다...")

link_ret = self.tee_record_pad.link(self.queue2_sink_pad)

# 4. 연결 결과 확인 및 자세한 로깅

if link_ret == Gst.PadLinkReturn.OK:

logger.info("녹화 경로가 성공적으로 연결되었습니다.")

self.is_recording = True

# 키프레임 요청 이벤트 전송 (녹화 시작 시 키프레임부터 시작하기 위함)

encoder_sink_pad = self.encoder.get_static_pad("sink")

if encoder_sink_pad:

# 키프레임 요청 이벤트 생성 및 전송

keyframe_event = Gst.Event.new_custom(

Gst.EventType.CUSTOM_DOWNSTREAM,

Gst.Structure.new_empty("GstForceKeyUnit")

)

encoder_sink_pad.send_event(keyframe_event)

logger.info("인코더에 키프레임 요청 이벤트를 전송했습니다.")

return True

else:

# 실패 원인에 따른 자세한 오류 메시지

error_msg = "알 수 없는 오류"

if link_ret == Gst.PadLinkReturn.WRONG_HIERARCHY:

error_msg = "잘못된 계층 구조"

elif link_ret == Gst.PadLinkReturn.WAS_LINKED:

error_msg = "이미 연결된 패드"

elif link_ret == Gst.PadLinkReturn.WRONG_DIRECTION:

error_msg = "잘못된 방향"

elif link_ret == Gst.PadLinkReturn.NOFORMAT:

error_msg = "호환되지 않는 형식"

elif link_ret == Gst.PadLinkReturn.NOSCHED:

error_msg = "스케줄링 문제"

elif link_ret == Gst.PadLinkReturn.REFUSED:

error_msg = "연결 거부됨"

logger.error(f"녹화 경로 연결 실패: {error_msg} (코드: {link_ret})")

return False

else:

logger.error("녹화에 필요한 패드가 준비되지 않았습니다.")

return False

except Exception as e:

logger.error(f"녹화 시작 중 오류 발생: {e}")

logger.error(traceback.format_exc())

return False

  

def stop_recording(self) -> bool:

"""

녹화 중지 - tee와 녹화 큐 사이의 링크를 해제

Returns:

bool: 녹화 중지 성공 여부

"""

if not self.is_recording:

logger.warning("녹화가 진행 중이 아닙니다.")

return True

try:

logger.info("녹화 중지: tee와 녹화 경로 연결을 해제합니다.")

# 녹화 중지를 위해 패드 언링크

if self.tee_record_pad and self.queue2_sink_pad:

# 1. 먼저 tee의 srcpad에 probe를 설정하여 추가 데이터가 흐르지 않도록 차단

logger.info("tee srcpad에 블록 프로브를 설정합니다.")

probe_id = self.tee_record_pad.add_probe(

Gst.PadProbeType.BLOCK_DOWNSTREAM,

self._pad_probe_block_callback,

None

)

# 2. probe가 활성화될 시간을 줍니다

time.sleep(0.2)

# 3. 파이프라인이 차단된 상태에서 안전하게 패드 언링크

if self.tee_record_pad.is_linked():

self.tee_record_pad.unlink(self.queue2_sink_pad)

logger.info("녹화 경로 패드가 안전하게 언링크되었습니다.")

# 4. 프로브 제거

self.tee_record_pad.remove_probe(probe_id)

logger.info("블록 프로브가 제거되었습니다.")

# 5. splitmuxsink에 전달하는 모든 버퍼가 중지된 후에 EOS 이벤트를 splitmuxsink에 전송

# 이는 파일이 올바르게 닫히고 finalize되도록 하기 위함입니다.

time.sleep(0.3) # 모든 버퍼가 처리될 시간을 주기

# 6. splitmuxsink에 EOS 전송 (녹화 경로가 언링크되어 있으므로 새 데이터는 없음)

# 하지만 파일을 적절히 닫기 위해 필요함

splitmux_sink_pad = self.splitmuxsink.get_static_pad("video")

if splitmux_sink_pad:

logger.info("splitmuxsink에 EOS 이벤트를 보내 파일을 정상 종료합니다.")

splitmux_sink_pad.send_event(Gst.Event.new_eos())

# 7. 파일이 완전히 저장될 시간을 주기

time.sleep(0.5)

# 녹화 중지 상태로 설정

self.is_recording = False

# 추가 정리 작업

logger.info("녹화가 성공적으로 중지되었습니다.")

return True

else:

logger.error("녹화 중지에 필요한 패드가 없습니다.")

return False

except Exception as e:

logger.error(f"녹화 중지 중 오류 발생: {e}")

logger.error(traceback.format_exc())

self.is_recording = False # 오류 발생해도 녹화 중지 상태로 설정

return False

  

def _pad_probe_block_callback(self, pad, probe_info, user_data):

"""

패드 프로브 콜백 - 패드를 차단하여 안전한 언링크를 가능하게 함

Args:

pad: 프로브가 설정된 패드

probe_info: 프로브 정보

user_data: 사용자 데이터

Returns:

Gst.PadProbeReturn: 프로브 반환 값

"""

logger.info("패드 프로브가 활성화되었습니다 - 스트림이 차단됨")

return Gst.PadProbeReturn.OK

  

def stop(self) -> bool:

"""

스트리밍 중지 (미리보기 및 녹화 모두 중지)

Returns:

bool: 중지 성공 여부

"""

if not GSTREAMER_AVAILABLE:

logger.error("GStreamer not available, cannot stop streaming")

return False

if not self.is_running:

logger.warning("Streaming is not running")

return True

try:

# 녹화 중이면 먼저 녹화 중지

if self.is_recording:

logger.info("Stopping recording before stopping the stream...")

self.stop_recording()

logger.info("Stopping streaming...")

# EOS 이벤트 전송

if not self.received_eos and self.tee:

sinkpad = self.tee.get_static_pad("sink")

if sinkpad:

eos_event = self.gst.create_eos_event()

if eos_event:

logger.info("Sending EOS event to pipeline")

sinkpad.send_event(eos_event)

# EOS 이벤트 처리 대기

for _ in range(30): # 최대 3초 대기

if self.received_eos:

break

time.sleep(0.1)

# 메인 루프 종료

if self.loop and hasattr(self.loop, 'is_running') and hasattr(self.loop, 'quit'):

if self.loop.is_running():

logger.info("Quitting GLib MainLoop")

self.loop.quit()

# 파이프라인을 PAUSED 상태로 전환

if self.pipeline:

logger.info("Setting pipeline to PAUSED state")

paused_state = self.gst.get_state_enum("PAUSED")

if paused_state:

ret = self.pipeline.set_state(paused_state)

logger.info(f"Pipeline state change to PAUSED returned: {self.gst.state_change_return_get_name(ret)}")

if ret == Gst.StateChangeReturn.ASYNC:

wait_ret = self.gst.wait_for_state_change(self.pipeline, 3 * Gst.SECOND)

logger.info(f"Wait for PAUSED state returned: {self.gst.state_change_return_get_name(wait_ret)}")

# 파이프라인을 NULL 상태로 변경

logger.info("Setting pipeline to NULL state")

null_state = self.gst.get_state_enum("NULL")

if null_state:

ret = self.pipeline.set_state(null_state)

logger.info(f"Pipeline state change to NULL returned: {self.gst.state_change_return_get_name(ret)}")

if ret == Gst.StateChangeReturn.ASYNC:

wait_ret = self.gst.wait_for_state_change(self.pipeline, 3 * Gst.SECOND)

logger.info(f"Wait for NULL state returned: {self.gst.state_change_return_get_name(wait_ret)}")

# 충분한 대기 시간

time.sleep(1.0)

# 파이프라인 및 관련 객체 해제 - 매우 중요

self.pipeline = None

self.source = None

self.jpeg_filter = None

self.jpegdec = None

self.converter = None

self.raw_fps_filter = None

self.raw_fmt_filter = None

self.encoder = None

self.parser = None

self.splitmuxsink = None

self.muxer = None

self.queue0 = None

self.queue1 = None

self.queue2 = None

self.tee_record_pad = None

self.queue2_sink_pad = None

self.is_running = False

self.is_recording = False

logger.info("Streaming stopped successfully")

return True

except Exception as e:

logger.error(f"Error stopping streaming: {e}")

self.is_running = False

self.is_recording = False

return False

  

def wait_for_keyboard_interrupt(self) -> None:

"""키보드 인터럽트(Ctrl+C)를 기다리는 메인 루프"""

signal.signal(signal.SIGINT, self.stop_handler)

logger.info("Press Ctrl+C to stop streaming")

try:

while self.is_running:

time.sleep(0.1)

except KeyboardInterrupt:

logger.info("KeyboardInterrupt received, stopping streaming gracefully...")

self.stop()

# 쓰레드가 정상적으로 종료될 때까지 대기

if self.loop_thread and self.loop_thread.is_alive():

logger.info("Waiting for GStreamer pipeline to finalize...")

self.loop_thread.join(timeout=5.0) # 최대 5초 대기 (파일 종료에 충분한 시간)

  

def stop_handler(self, sig: int, frame: Any) -> None:

"""시그널 핸들러"""

logger.info(f"Received signal {sig}, stopping streaming gracefully...")

self.stop()

  

def cleanup(self) -> None:

"""리소스 정리"""

try:

# 녹화 중이면 중지

if self.is_recording:

logger.info("Cleaning up: stopping active recording")

self.stop_recording()

# 스트리밍 중이면 중지

if self.is_running:

logger.info("Cleaning up: stopping active streaming")

self.stop()

# 메인 루프 종료

if self.loop and hasattr(self.loop, 'is_running') and hasattr(self.loop, 'quit'):

if self.loop.is_running():

logger.info("Quitting GLib MainLoop")

self.loop.quit()

# 쓰레드가 정상적으로 종료될 때까지 대기

if self.loop_thread and self.loop_thread.is_alive():

logger.info("Waiting for GLib MainLoop thread to finish")

self.loop_thread.join(timeout=3.0) # 최대 3초 대기

# 파이프라인이 남아있으면 정리

if self.pipeline:

logger.info("Cleaning up remaining pipeline")

null_state = self.gst.get_state_enum("NULL")

if null_state:

self.pipeline.set_state(null_state)

# 상태 변경 완료 대기

self.gst.wait_for_state_change(self.pipeline, 2 * Gst.SECOND)

# 파이프라인 및 요소들 참조 정리

self.pipeline = None

self.source = None

self.jpeg_filter = None

self.jpegdec = None

self.converter = None

self.raw_fps_filter = None

self.raw_fmt_filter = None

self.encoder = None

self.parser = None

self.splitmuxsink = None

self.muxer = None

self.queue0 = None

self.queue1 = None

self.queue2 = None

self.tee_record_pad = None

self.queue2_sink_pad = None

# GStreamer 메모리 정리 (선택적)

# 여기서 GC를 강제할 수 있지만, Python의 GC가 일반적으로 처리함

import gc

gc.collect()

logger.info("All resources cleaned up")

except Exception as e:

logger.error(f"Error during cleanup: {e}")

logger.error(traceback.format_exc())

  

def timestamp_probe_callback(self, pad, info, *args):

"""

타임스탬프 관리를 위한 패드 프로브 콜백

Args:

pad: GStreamer 패드

info: 프로브 정보

Returns:

Gst.PadProbeReturn: 프로브 반환 값

"""

buffer = info.get_buffer()

if buffer:

# 버퍼의 타임스탬프 정보 가져오기

pts = buffer.pts

dts = buffer.dts

duration = buffer.duration

# 현재 버퍼의 타임스탬프 정보 출력

#logger.info(f"Buffer timestamp - PTS: {pts/Gst.SECOND:.3f}초, DTS: {dts/Gst.SECOND if dts != Gst.CLOCK_TIME_NONE else 'NONE'}초, Duration: {duration/Gst.SECOND if duration != Gst.CLOCK_TIME_NONE else 'NONE'}초")

if pts != Gst.CLOCK_TIME_NONE:

# 새 파일 세그먼트가 시작되었는지 확인

time_in_segment = pts - self.base_time

if time_in_segment >= 60 * Gst.SECOND: # 1분(60초) 초과

# 새 세그먼트 시작

self.current_segment += 1

logger.info(f"새 파일 세그먼트 시작: {self.current_segment}, 타임스탬프 초기화")

  

# 새 세그먼트 이벤트 생성

#segment = Gst.Segment()

#segment.init(Gst.Format.TIME)

#segment.start = 0

#segment.stop = Gst.CLOCK_TIME_NONE

#segment.time = self.base_time

#segment.position = 0

#segment.base = self.base_time

#segment.offset = self.base_time

# 세그먼트 이벤트 전송

#segment_event = Gst.Event.new_segment(segment)

#result = pad.push_event(segment_event)

#logger.info(f"새 세그먼트 이벤트 전송 결과: {result}")

  

# base_time 갱신 - 10초 오차 보정

# 첫 세그먼트(current_segment=1)에서는 보정 없이, 그 이후부터 10초 보정

if self.current_segment == 1:

self.base_time = pts

else:

# 두 번째 파일부터는 10초 오차 보정

self.base_time = pts - 10 * Gst.SECOND

logger.info(f"두 번째 이후 파일: 10초 오차 보정 적용")

# 현재 세그먼트 내의 상대적 타임스탬프 계산

#logger.info(f"time_in_segment: {time_in_segment}, base_time: {self.base_time/Gst.SECOND:.3f}")

adjusted_pts = pts - self.base_time

buffer.pts = adjusted_pts

#logger.info(f"Buffer timestamp - PTS: {buffer.pts/Gst.SECOND:.3f}초, DTS: {buffer.dts/Gst.SECOND:.3f}초")

# 디버깅 (100초마다 로깅)

if adjusted_pts % (100 * Gst.SECOND) < 100 * 1000000:

logger.info(f"타임스탬프 조정: 원본={pts/Gst.SECOND:.2f}초, 조정={adjusted_pts/Gst.SECOND:.2f}초, 세그먼트={self.current_segment}")

  

return Gst.PadProbeReturn.OK

  

def take_snapshot(self, filename: Optional[str] = None) -> bool:

"""

현재 화면의 스냅샷을 캡처하여 저장합니다.

Args:

filename: 저장할 파일 이름 (None이면 타임스탬프 기반으로 자동 생성)

Returns:

bool: 스냅샷 요청 성공 여부

"""

if not self.is_running:

logger.error("파이프라인이 실행 중이 아닙니다. 스냅샷을 촬영할 수 없습니다.")

return False

try:

# 파일 이름이 제공되지 않으면 타임스탬프 기반으로 생성

if filename is None:

from datetime import datetime

timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")

filename = f"snapshot_{timestamp}.jpg"

logger.info(f"스냅샷 캡처 요청: {filename}")

# 스냅샷 싱크로 새 이미지 요청

self.snapshot_pending = True

self.snapshot_filename = filename

# 실제 캡처 및 저장은 _on_snapshot_handoff 콜백에서 처리됨

return True

except Exception as e:

logger.error(f"스냅샷 요청 중 오류 발생: {e}")

logger.error(traceback.format_exc())

return False

  
  

def verify_environment() -> Dict[str, bool]:

"""환경 검증 및 필요한 패키지 확인"""

results = {

"gstreamer_available": GSTREAMER_AVAILABLE,

"camera_device_exists": os.path.exists("/dev/video0"),

"output_directory_writable": os.access(".", os.W_OK),

}

# 결과 출력

for check, result in results.items():

logger.info(f"Environment check: {check} = {result}")

return results

  

def legacy_main() -> int:

"""

레거시 메인 함수 (기존 방식)

Returns:

int: 종료 코드 (0: 성공, 1: 실패)

"""

try:

# 환경 확인

env_check = verify_environment()

if not env_check["gstreamer_available"]:

logger.error("GStreamer is not available. Cannot proceed with video recording.")

return 1

if not env_check["camera_device_exists"]:

logger.warning("Camera device not found at /dev/video0. Will try to use it anyway.")

# 비디오 레코더 생성 및 시작

recorder = VideoRecorder()

# 레코딩 시작

if not recorder.start():

logger.error("Failed to start recording")

return 1

logger.info("Recording started. Press Ctrl+C to stop.")

# 키보드 인터럽트(Ctrl+C) 대기

recorder.wait_for_keyboard_interrupt()

# 프로그램 종료

logger.info("Exiting")

return 0

except KeyboardInterrupt:

logger.info("Keyboard interrupt received, exiting")

return 0

except Exception as e:

logger.error(f"Unhandled exception: {e}")

return 1

  

def multithreaded_main() -> int:

"""

멀티스레드 아키텍처 모드 진입점

AppManager를 사용하여 애플리케이션을 실행합니다.

Returns:

int: 종료 코드 (0: 성공, 1: 실패)

"""

try:

# 환경 검증

env_check = verify_environment()

if not env_check["gstreamer_available"]:

logger.error("GStreamer is not available. Cannot proceed.")

return 1

if not env_check["camera_device_exists"]:

logger.warning("Camera device not found. Application may not function correctly.")

# 실패하더라도 계속 진행, 런타임에 카메라가 연결될 수 있음

if not env_check["output_directory_writable"]:

logger.error("Output directory is not writable. Cannot proceed.")

return 1

# core 패키지에서 AppManager 임포트

try:

# 경로 설정이 되어 있지 않으면 현재 디렉토리 추가

current_dir = os.path.dirname(os.path.abspath(__file__))

sys.path.append(current_dir)

# 패키지 임포트

try:

from core.app_manager import AppManager

logger.info("Using multi-threaded architecture with AppManager")

except (ImportError, ModuleNotFoundError) as e:

logger.error(f"Failed to import AppManager: {e}")

logger.error("Falling back to legacy mode")

return legacy_main()

except Exception as e:

logger.error(f"Unexpected error importing modules: {e}")

logger.error("Falling back to legacy mode")

return legacy_main()

# AppManager 생성 및 실행

try:

app_manager = AppManager()

return app_manager.run()

except Exception as e:

logger.error(f"Error running AppManager: {e}")

import traceback

traceback.print_exc()

logger.error("Attempting to fall back to legacy mode")

return legacy_main()

except Exception as e:

logger.error(f"Unhandled exception in multithreaded_main: {e}")

import traceback

traceback.print_exc()

return 1

  

def main() -> int:

"""

메인 함수

환경 변수에 따라 멀티스레드 아키텍처 또는 레거시 모드로 실행합니다.

Returns:

int: 종료 코드 (0: 성공, 1: 실패)

"""

# 멀티스레드 아키텍처 사용 여부 확인

if USE_MULTITHREADED_ARCHITECTURE:

logger.info("Starting in multi-threaded architecture mode")

return multithreaded_main()

else:

logger.info("Starting in legacy mode")

return legacy_main()

  

if __name__ == "__main__":

# 프로그램 시작

logger.info("Camera App starting...")

exit_code = main()

sys.exit(exit_code)
```
